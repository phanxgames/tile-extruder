# Technical Specification

## Architecture Overview

### Design Pattern
**Single-Module Console Application** using procedural programming paradigm.

**Module**: `Module1.vb`
- Entry Point: `Main()`
- Helper Functions: `PrintUsage()`, `PrintHelp()`, `ArgLine()`, `ParseArg()`
- Core Logic: `Chooch()`

### Component Diagram
```
┌─────────────────────────────────────────┐
│         TileExtruder.exe                │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │         Module1 (Main)            │ │
│  │                                   │ │
│  │  ├─ Main()                        │ │
│  │  │   ├─ Argument Parsing          │ │
│  │  │   ├─ User Input                │ │
│  │  │   └─ Call Chooch()             │ │
│  │  │                                │ │
│  │  ├─ Chooch()                      │ │
│  │  │   ├─ Image Loading             │ │
│  │  │   ├─ Bitmap Creation           │ │
│  │  │   ├─ Graphics Operations       │ │
│  │  │   └─ File Saving               │ │
│  │  │                                │ │
│  │  └─ Helper Functions              │ │
│  │      ├─ PrintUsage()              │ │
│  │      ├─ PrintHelp()               │ │
│  │      ├─ ArgLine()                 │ │
│  │      └─ ParseArg()                │ │
│  └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
           │                  │
           │                  │
    ┌──────▼──────┐    ┌─────▼─────┐
    │ System.     │    │  System.  │
    │ Drawing     │    │  IO       │
    └─────────────┘    └───────────┘
```

## Technology Stack

### Framework
- **.NET Framework 4.0** (minimum)
- Target: AnyCPU (32-bit or 64-bit compatible)
- Output Type: Console Executable (.exe)

### Dependencies

#### System Assemblies
1. **System.Drawing.dll**
   - Purpose: Image manipulation via GDI+
   - Key Classes: `Bitmap`, `Graphics`, `Color`, `Rectangle`, `SolidBrush`
   - Critical for: Loading, processing, saving images

2. **System.dll**
   - Purpose: Core system functionality
   - Key Classes: `Console`, `Process`

3. **System.IO** (via System.dll)
   - Purpose: File system operations
   - Key Classes: `File`, `Path`, `FileInfo`

4. **System.Xml.dll**
   - Purpose: Project dependency (not actively used in code)

5. **System.Data.dll**
   - Purpose: Project dependency (not actively used in code)

### VB.NET Specific Features

#### Imports
```vb
Imports System.Drawing
```

#### My Namespace Usage
```vb
My.Application.CommandLineArgs      ' Command-line arguments collection
My.Application.Info.Version         ' Application version info
My.Computer.FileSystem.GetFileInfo  ' File information retrieval
```

The `My` namespace is VB.NET-specific and provides simplified access to .NET Framework features.

## Data Structures

### Module-Level Variables (State)

```vb
' File paths
Dim input As String          ' Input file path
Dim output As String         ' Output file path

' Tile dimensions
Dim TileW As Integer         ' Tile width in pixels
Dim TileH As Integer         ' Tile height in pixels

' Padding and spacing (initialized to -1 to detect missing values)
Dim xPad As Integer = -1     ' Horizontal padding
Dim yPad As Integer          ' Vertical padding
Dim xSpace As Integer = -1   ' Horizontal spacing
Dim ySpace As Integer        ' Vertical spacing

' Processing mode
Dim diagProcessLevel As Integer = 3  ' Diagonal corner fill mode (0-3)
```

### Function Parameters and Return Types

#### ParseArg()
```vb
Function ParseArg(arg As String) As String()
```
- **Input**: Single command-line argument string
- **Output**: String array [name, xValue, yValue]

#### Chooch()
```vb
Sub Chooch()
```
- **Input**: Uses module-level variables
- **Output**: Writes file to disk, no return value

#### PrintUsage(), PrintHelp(), ArgLine()
```vb
Sub PrintUsage()
Sub PrintHelp()
Sub ArgLine(name As String, desc As String, Optional extraBlank As Boolean = True)
```
- **Input**: Text parameters for display
- **Output**: Console output, no return value

### GDI+ Objects

#### Bitmap
```vb
Dim b As New Bitmap(input)   ' Input bitmap
Dim o As New Bitmap(width, height)  ' Output bitmap
```

Properties used:
- `Width`, `Height`: Dimensions
- `HorizontalResolution`, `VerticalResolution`: DPI
- Methods: `GetPixel()`, `SetResolution()`, `Save()`

#### Graphics
```vb
Dim g As Graphics = Graphics.FromImage(o)
```

Methods used:
- `DrawImage()`: Copy/transform image regions
- `FillRectangle()`: Fill solid color areas
- `Dispose()`: Release GDI+ resources

#### Rectangle
```vb
Dim srcRect As New Rectangle(x, y, width, height)
```

Used for defining source and destination regions.

#### Color
```vb
Dim fill As Color
fill = b.GetPixel(x, y)
fill = Color.FromArgb(a, r, g, b)
```

ARGB color structure (32-bit).

#### SolidBrush
```vb
New SolidBrush(color)
```

Used for filling rectangles with solid colors.

## Algorithms and Complexity

### Time Complexity

#### Main Loop
```
O(tilesX * tilesY * operations)
```

Where:
- `tilesX = inputWidth / tileWidth`
- `tilesY = inputHeight / tileHeight`
- `operations` = constant time per tile (DrawImage, FillRectangle)

**Overall**: O(n) where n = number of tiles

#### Diagonal Mode 3 (Worst Case)
```
O(tilesX * tilesY * 4)  // 4 FillRectangle calls per tile
```

Still O(n) with higher constant factor.

### Space Complexity

```
O(inputSize + outputSize)
```

Where:
- `inputSize = inputWidth * inputHeight * 4 bytes` (32bpp)
- `outputSize = outputWidth * outputHeight * 4 bytes` (32bpp)

**Additional**: Graphics context and temporary objects (negligible).

## File I/O Specifications

### Input File Format
- **Supported**: Any raster format supported by GDI+ (PNG, BMP, JPG, GIF, TIFF)
- **Recommended**: PNG (supports transparency/alpha channel)
- **Constraint**: Dimensions must be exact multiples of tile size

### Output File Format
- **Format**: Same as input (based on extension)
- **Bit Depth**: Always 32bpp ARGB
- **Compression**: Format-specific (PNG uses lossless compression)
- **DPI**: Matches input DPI

### File Naming Convention
**Auto-generated output**:
```
Input:  "tileset.png"
Output: "tileset_extruded.png"

Input:  "path/to/tiles.bmp"
Output: "path/to/tiles_extruded.bmp"
```

**Pattern**: `<directory>/<basename>_extruded<extension>`

## Image Processing Details

### GDI+ DrawImage() with Negative Dimensions

This is the key technique for "bidi" (bidirectional) mirroring:

```vb
' Normal draw
g.DrawImage(source, destRect, srcRect, GraphicsUnit.Pixel)

' Horizontal flip (negative width)
g.DrawImage(source, New Rectangle(x, y, -width, height), srcRect, GraphicsUnit.Pixel)

' Vertical flip (negative height)
g.DrawImage(source, New Rectangle(x, y, width, -height), srcRect, GraphicsUnit.Pixel)
```

**Behavior**: Negative dimensions cause GDI+ to mirror the image along that axis.

### Pixel Addressing

```vb
' Get pixel at specific coordinate
Dim color As Color = bitmap.GetPixel(x, y)

' Coordinates are 0-based
' (0, 0) = top-left
' (width-1, height-1) = bottom-right
```

### Color Averaging Algorithm

```vb
' Integer division for averaging
Dim avgChannel As Integer = (val1 + val2 + val3 + val4) / 4

' Creates Color from ARGB
Dim averaged As Color = Color.FromArgb(avgA, avgR, avgG, avgB)
```

**Note**: Uses integer division (truncates decimals).

## Console Interface

### Color Scheme
```vb
Console.ForegroundColor = ConsoleColor.Cyan      ' "Usage:"
Console.ForegroundColor = ConsoleColor.White     ' Highlighted text
Console.BackgroundColor = ConsoleColor.DarkBlue  ' Argument examples
Console.ForegroundColor = ConsoleColor.Green     ' X values
Console.ForegroundColor = ConsoleColor.Red       ' Y values
```

### User Input
```vb
' Read line input
Console.ReadLine()

' Read single key
Console.ReadKey()  ' Returns ConsoleKeyInfo
```

## Build Configuration

### Debug Build
- Platform: AnyCPU
- Debug Symbols: Full
- Optimizations: Disabled
- Output: `bin/Debug/TileExtruder.exe`

### Release Build
- Platform: AnyCPU
- Debug Symbols: PDB only
- Optimizations: Enabled
- Output: `bin/Release/TileExtruder.exe`

### Project Settings
```xml
<TargetFrameworkVersion>v4.0</TargetFrameworkVersion>
<OutputType>Exe</OutputType>
<StartupObject>TileExtruder.Module1</StartupObject>
<RootNamespace>TileExtruder</RootNamespace>
<AssemblyName>TileExtruder</AssemblyName>
```

## Resource Files

### Application Icon
- File: `extruder.ico`
- Embedded: Yes
- Purpose: Executable icon in Windows Explorer

### Assembly Information
Location: `My Project/AssemblyInfo.vb`
Contains:
- Assembly title
- Description
- Company
- Product name
- Copyright
- Version information

## Limitations and Constraints

### Technical Limitations

1. **Platform Dependency**
   - Windows-only due to System.Drawing/GDI+
   - .NET Framework required (not .NET Core/.NET 5+)

2. **Memory Constraints**
   - Entire input and output bitmaps loaded in RAM
   - Large tilesets may cause memory issues
   - 32bpp format doubles memory usage for 16bpp inputs

3. **GDI+ Limitations**
   - Thread-unsafe (single-threaded processing)
   - No GPU acceleration
   - Relatively slow for large images

4. **Precision**
   - Integer division for tile counts (no partial tiles)
   - Integer averaging (color precision loss)
   - No dithering or advanced color quantization

### Known Bugs

1. **Pad > Tile Size**
   - Setting padding larger than tile dimensions produces incorrect results
   - No validation or warning
   - Recommendation: Add validation in future version

2. **Forced 32bpp Output**
   - All output converted to 32bpp regardless of input
   - Increases file size for palettized or 16bpp inputs
   - Recommendation: Preserve input bit depth

## Performance Characteristics

### Typical Performance
- Small tileset (256x256, 16x16 tiles): < 1 second
- Medium tileset (2048x2048, 32x32 tiles): 2-5 seconds
- Large tileset (4096x4096, 16x16 tiles): 10-30 seconds

**Factors**:
- Tile count (dominant factor)
- Padding amount (more padding = more operations)
- Diagonal mode (mode 3 is slowest)
- Output format (PNG compression is slower than BMP)

### Optimization Opportunities
1. Use `LockBits()` for faster pixel access (commented code exists)
2. Multi-threading (process tiles in parallel)
3. GPU acceleration (OpenGL, DirectX, or compute shaders)
4. Incremental processing (stream-based for large images)

## Security Considerations

### File Path Handling
- No sanitization of input/output paths
- Vulnerable to path traversal (not a concern for CLI tool)
- Overwrites files without backup

### Input Validation
- Limited validation of image files
- Relies on GDI+ to handle corrupt images
- May crash on invalid image data

### Recommendations for Production
1. Add file path sanitization
2. Validate image dimensions before processing
3. Add try-catch around bitmap operations
4. Implement size limits for DOS prevention
