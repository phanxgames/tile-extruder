# API Specification

## Command-Line Interface Specification

### Syntax
```
TileExtruder <input_file> [options...] [output_file]
```

### Arguments

#### Positional Arguments

##### Input File (Required)
- **Position**: First argument
- **Type**: String (file path)
- **Description**: Path to input image file
- **Validation**: Must exist, must be readable image file
- **Example**: `tileset.png`

##### Output File (Optional)
- **Position**: Last argument (if doesn't contain '=')
- **Type**: String (file path)
- **Description**: Path to output image file
- **Default**: `<input_directory>/<input_basename>_extruded<input_extension>`
- **Example**: `output.png`

#### Named Arguments

##### size
- **Format**: `size=<width>[,<height>]`
- **Type**: Integer(s), pixels
- **Range**: > 0
- **Required**: Yes (prompted if missing)
- **Default**: None
- **Description**: Dimensions of individual tiles in pixels
- **Behavior**: If only one value provided, used for both width and height
- **Examples**:
  - `size=16` → 16x16 tiles
  - `size=32,16` → 32px wide, 16px tall tiles

##### pad
- **Format**: `pad=<x>[,<y>]`
- **Type**: Integer(s), pixels
- **Range**: >= 0
- **Required**: Yes (prompted if missing)
- **Default**: None
- **Description**: Amount of pixel extrusion on each side of tiles
- **Behavior**: If only one value provided, used for both x and y
- **Examples**:
  - `pad=2` → 2px padding on all sides
  - `pad=3,1` → 3px horizontal, 1px vertical padding

##### space
- **Format**: `space=<x>[,<y>]`
- **Type**: Integer(s), pixels
- **Range**: >= 0
- **Required**: Yes (prompted if missing)
- **Default**: None
- **Description**: Empty space between tiles
- **Behavior**: If only one value provided, used for both x and y
- **Examples**:
  - `space=0` → No spacing
  - `space=2,1` → 2px horizontal, 1px vertical spacing

##### diag
- **Format**: `diag=<mode>`
- **Type**: Integer
- **Range**: 0-3
- **Required**: No
- **Default**: 3
- **Description**: Diagonal corner pixel fill mode
- **Values**:
  - `0`: No diagonal fill (corners undefined)
  - `1`: Fill with first pixel (top-left corner color)
  - `2`: Fill with average of four corner pixels
  - `3`: Extrude each corner diagonally (quadrant-based)
- **Example**: `diag=2`

#### Flags

##### -h, -help
- **Format**: `-h` or `-help`
- **Type**: Boolean flag
- **Description**: Display help information and exit
- **Behavior**: Shows version, usage, and detailed parameter descriptions
- **Example**: `TileExtruder -h`

### Return Codes

| Code | Description |
|------|-------------|
| 0    | Success |
| 1    | Error (various, see error messages) |

**Note**: Current implementation uses `End` statement which may not set proper exit codes. Recommendation: Use `Environment.Exit(code)` in migration.

## Function Reference

### Main Entry Point

#### Main()
```vb
Sub Main()
```

**Description**: Application entry point

**Flow**:
1. Check for arguments
2. Parse help flag
3. Validate input file
4. Parse arguments
5. Prompt for missing parameters
6. Execute processing
7. Display results

**Side Effects**: Console output, file I/O

### Helper Functions

#### ParseArg()
```vb
Function ParseArg(arg As String) As String()
```

**Parameters**:
- `arg`: Command-line argument string

**Returns**: String array `[name, xValue, yValue]`

**Behavior**:
- Splits by `=` and `,`
- Returns 3+ elements: `[name, x, y, ...]`
- Returns 2 elements: `[name, value, value]` (duplicates value)
- Returns 1 element: `[value, "0", "0"]`

**Examples**:
```vb
ParseArg("size=16,32") → ["size", "16", "32"]
ParseArg("pad=2")      → ["pad", "2", "2"]
ParseArg("input.png")  → ["input.png", "0", "0"]
```

#### Chooch()
```vb
Sub Chooch()
```

**Description**: Core image processing function

**Inputs** (via module variables):
- `input`: Input file path
- `output`: Output file path
- `TileW`, `TileH`: Tile dimensions
- `xPad`, `yPad`: Padding amounts
- `xSpace`, `ySpace`: Spacing amounts
- `diagProcessLevel`: Diagonal fill mode

**Outputs**:
- Creates output image file
- Console status messages

**Algorithm**:
1. Load input bitmap
2. Calculate tile grid and output dimensions
3. Create output bitmap with proper DPI
4. For each tile:
   - Apply diagonal fill
   - Copy source tile
   - Extrude edges
5. Save output file (with overwrite check)

**Exceptions**:
- File I/O errors (uncaught)
- Out of memory (uncaught)
- Invalid image format (uncaught)

#### PrintUsage()
```vb
Sub PrintUsage()
```

**Description**: Display command-line usage syntax

**Output**:
```
Usage: TileExtruder.exe inputfile [args] [outputfile]
```

**Colors**: Cyan for "Usage:", White for executable name

#### PrintHelp()
```vb
Sub PrintHelp()
```

**Description**: Display detailed help information

**Output**: Formatted list of all parameters with descriptions

**Colors**:
- White: Parameter names
- Background DarkBlue: Argument format examples
- Green: X values
- Red: Y values

#### ArgLine()
```vb
Sub ArgLine(name As String, desc As String, Optional extraBlank As Boolean = True)
```

**Parameters**:
- `name`: Parameter name
- `desc`: Description text
- `extraBlank`: Whether to add extra blank line (default: True)

**Description**: Helper for formatting help output lines

**Output**: Tab-indented parameter line with description

## Data Types and Structures

### Module-Level Variables

```vb
' String types
Dim input As String              ' Input file path
Dim output As String             ' Output file path

' Integer types
Dim TileW As Integer             ' Tile width (>= 1)
Dim TileH As Integer             ' Tile height (>= 1)
Dim xPad As Integer = -1         ' Horizontal padding (>= 0, -1 = not set)
Dim yPad As Integer              ' Vertical padding (>= 0)
Dim xSpace As Integer = -1       ' Horizontal spacing (>= 0, -1 = not set)
Dim ySpace As Integer            ' Vertical spacing (>= 0)
Dim diagProcessLevel As Integer = 3  ' Diagonal mode (0-3)
```

### GDI+ Types Used

#### Bitmap
```vb
Class: System.Drawing.Bitmap
```

**Constructor**:
```vb
New Bitmap(path As String)           ' Load from file
New Bitmap(width As Integer, height As Integer)  ' Create blank
```

**Methods Used**:
- `GetPixel(x, y) As Color`: Get pixel color at coordinates
- `SetResolution(dpiX, dpiY)`: Set image resolution
- `Save(path As String)`: Save to file

**Properties Used**:
- `Width As Integer`: Image width
- `Height As Integer`: Image height
- `HorizontalResolution As Single`: DPI horizontal
- `VerticalResolution As Single`: DPI vertical

#### Graphics
```vb
Class: System.Drawing.Graphics
```

**Factory Method**:
```vb
Graphics.FromImage(bitmap As Bitmap) As Graphics
```

**Methods Used**:
- `DrawImage(source, destRect, srcRect, unit)`: Copy/transform image region
- `FillRectangle(brush, x, y, width, height)`: Fill solid color
- `Dispose()`: Release resources

#### Color
```vb
Structure: System.Drawing.Color
```

**Factory Method**:
```vb
Color.FromArgb(alpha, red, green, blue) As Color
```

**Properties**:
- `A As Byte`: Alpha channel (0-255)
- `R As Byte`: Red channel (0-255)
- `G As Byte`: Green channel (0-255)
- `B As Byte`: Blue channel (0-255)

#### Rectangle
```vb
Structure: System.Drawing.Rectangle
```

**Constructor**:
```vb
New Rectangle(x As Integer, y As Integer, width As Integer, height As Integer)
```

**Properties**:
- `X As Integer`: Left coordinate
- `Y As Integer`: Top coordinate
- `Width As Integer`: Width
- `Height As Integer`: Height

#### SolidBrush
```vb
Class: System.Drawing.SolidBrush
```

**Constructor**:
```vb
New SolidBrush(color As Color)
```

## Calculation Formulas

### Output Dimensions
```
tilesX = floor(inputWidth / tileWidth)
tilesY = floor(inputHeight / tileHeight)

outputWidth = tilesX × (tileWidth + 2×xPad + 2×xSpace)
outputHeight = tilesY × (tileHeight + 2×yPad + 2×ySpace)
```

### Tile Source Rectangle
```
For tile at grid position (tileX, tileY):

srcRect.X = tileX × tileWidth
srcRect.Y = tileY × tileHeight
srcRect.Width = tileWidth
srcRect.Height = tileHeight
```

### Tile Destination Position
```
For tile at grid position (tileX, tileY):

destX = tileX × (tileWidth + 2×xPad + 2×xSpace) + xSpace + xPad
destY = tileY × (tileHeight + 2×yPad + 2×ySpace) + ySpace + yPad
```

### Corner Fill Dimensions (Mode 3)
```
halfW = floor(tileWidth / 2)
halfH = floor(tileHeight / 2)

cornerWidth = halfW + 2×xPad
cornerHeight = halfH + 2×yPad
```

### Average Color Calculation (Mode 2)
```
avgAlpha = floor((c1.A + c2.A + c3.A + c4.A) / 4)
avgRed   = floor((c1.R + c2.R + c3.R + c4.R) / 4)
avgGreen = floor((c1.G + c2.G + c3.G + c4.G) / 4)
avgBlue  = floor((c1.B + c2.B + c3.B + c4.B) / 4)
```

**Note**: Integer division truncates decimal

## Algorithm Complexity

### Time Complexity

**Overall**: O(tilesX × tilesY × operations)

**Per-Tile Operations**:
- Diagonal fill: O(1) for modes 0-2, O(4) for mode 3 (constant)
- Tile copy: O(1) (GDI+ operation)
- Edge extrusion: O(4) (4 DrawImage calls)

**Total**: O(n) where n = number of tiles

**Dominated by**: Tile count, not padding/spacing amounts

### Space Complexity

**Memory Usage**:
```
inputMemory = inputWidth × inputHeight × 4 bytes
outputMemory = outputWidth × outputHeight × 4 bytes

total = inputMemory + outputMemory + overhead
```

**Overhead**: Graphics context, temporary objects (~1-10 MB)

**Example**:
```
Input: 1024×1024 = 1,048,576 pixels × 4 = 4,194,304 bytes (4 MB)
Output: 1408×1408 = 1,982,464 pixels × 4 = 7,929,856 bytes (7.6 MB)
Total: ~12 MB
```

## Error Conditions

### Input Validation Errors

#### File Not Found
- **Trigger**: Input file doesn't exist
- **Message**: `"Error: Input file does not exist."`
- **Action**: Exit program

#### Invalid Parameter
- **Trigger**: Exception during argument parsing
- **Message**: `"Error: Invalid parameter for argument '<name>'"`
- **Action**: Exit program

### File I/O Errors (Not Explicitly Handled)

#### Permission Denied
- **Trigger**: No read access to input or write access to output
- **Behavior**: Uncaught exception, program crash

#### Invalid Image Format
- **Trigger**: Input file not a valid image
- **Behavior**: Uncaught exception during Bitmap constructor

#### Disk Full
- **Trigger**: Insufficient space for output file
- **Behavior**: Uncaught exception during Save()

### Memory Errors (Not Handled)

#### Out of Memory
- **Trigger**: Image too large for available RAM
- **Behavior**: Uncaught exception, program crash

## Output File Format

### File Format
- **Type**: Same as input (PNG, BMP, JPG, etc.)
- **Determined by**: File extension of output path
- **Bit Depth**: Always 32bpp ARGB
- **Compression**: Format-specific default

### Metadata Preservation
- **DPI**: Copied from input (HorizontalResolution, VerticalResolution)
- **Color Profile**: Not explicitly preserved (GDI+ default handling)
- **EXIF Data**: Lost (not copied by GDI+)

### File Naming
**Auto-generated format**:
```
<directory>/<basename>_extruded<extension>
```

**Examples**:
```
Input: "tileset.png"        → Output: "tileset_extruded.png"
Input: "assets/map.bmp"     → Output: "assets/map_extruded.bmp"
Input: "/tmp/tiles.jpg"     → Output: "/tmp/tiles_extruded.jpg"
```

## Console Output Specification

### Normal Execution
```
Processing...
Wrote <bytes> bytes.
```

### No Arguments
```
Usage: TileExtruder.exe inputfile [args] [outputfile]
```

### Help Flag
```
TileExtruder <version> by Nobuyuki.
Get the latest version at https://github.com/nobuyukinyuu/tile-extruder

Usage: TileExtruder.exe inputfile [args] [outputfile]

All args can take 1 or 2 values in px, and are in the form arg1=x,y:
	size	Size of an individual tile in this set.

	pad	    Amount each tile is looped/extruded past its border.

	space	Amount of empty space between each tile.

	diag	Diagonal pixel extrusion fill mode. (default: 3)
	        0: No fill;
	        1: Use first pixel;
	        2: Average corner pixels.
	        3: Extrude corners diagonally
```

### Interactive Prompts
```
Tile X Size: <user_input>
Tile Y Size: <user_input>

X Padding: <user_input>
Y Padding: <user_input>

X Spacing: <user_input>
Y Spacing: <user_input>
```

### File Overwrite Prompt
```
<filename> already exists.  Overwrite? (y/n): <user_input>
```

**If 'y'**:
```
Processing...
Wrote <bytes> bytes.
```

**If not 'y'**:
```
Operation aborted.
```

### Error Messages
```
Error:  Input file does not exist.
```
```
Error: Invalid parameter for argument "<name>"
```

## Compatibility Requirements

### Input Compatibility
- **Image Formats**: Any supported by System.Drawing (PNG, BMP, JPG, GIF, TIFF, etc.)
- **Bit Depths**: Any (8bpp indexed, 16bpp, 24bpp, 32bpp)
- **Color Modes**: RGB, RGBA, Indexed, Grayscale
- **Transparency**: Preserved in output

### Output Compatibility
- **Bit Depth**: Always 32bpp ARGB
- **Transparency**: Fully supported
- **Maximum Dimensions**: Limited by System.Drawing (2^16 pixels per dimension)

### Platform Compatibility (Current)
- **OS**: Windows only (GDI+ dependency)
- **.NET**: Framework 4.0 or higher
- **Architecture**: AnyCPU (x86, x64, ARM with .NET support)

## Extension Points for Migration

### Recommended Additions
1. **Progress Callback**: Report processing progress
2. **Validation Hooks**: Pre-validate parameters
3. **Custom Output Format**: Specify bit depth, compression
4. **Batch Mode**: Process multiple files
5. **Config File**: Load parameters from JSON/YAML
6. **Logging**: Optional verbose output

### API Design Suggestion (for library version)
```pseudo
class TileExtruder:
    constructor(config: Config)

    function process(inputPath: String, outputPath: String) -> Result
    function processImage(inputImage: Image) -> Image
    function validate(config: Config) -> ValidationResult

    // Events
    onProgress(callback: (percent: Float) -> Void)
    onError(callback: (error: Error) -> Void)
```

**Example Usage**:
```pseudo
extruder = new TileExtruder({
    tileWidth: 16,
    tileHeight: 16,
    padX: 2,
    padY: 2,
    spaceX: 1,
    spaceY: 1,
    diagMode: 3
})

extruder.onProgress((percent) => {
    print("Processing: " + percent + "%")
})

result = extruder.process("input.png", "output.png")
if result.success:
    print("Success! Wrote " + result.fileSize + " bytes")
else:
    print("Error: " + result.error)
```
