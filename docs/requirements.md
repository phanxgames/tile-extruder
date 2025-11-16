# Requirements Specification

## Functional Requirements

### FR1: Command-Line Interface
**Priority**: Critical
**Description**: Application must operate as a command-line tool accepting arguments and displaying output to console.

**Inputs**:
- Input file path (required, first argument)
- Output file path (optional, last argument if doesn't contain '=')
- Named parameters (optional)

**Outputs**:
- Processed image file
- Console status messages
- File size confirmation

### FR2: Image Processing Parameters

#### FR2.1: Tile Size (`size=x,y`)
- Specifies width and height of individual tiles in pixels
- Required parameter (prompted if not provided)
- Must support both dimensions (can be same: `size=16` → `size=16,16`)

#### FR2.2: Padding (`pad=x,y`)
- Amount each tile is extruded past its border in pixels
- Required parameter (prompted if not provided)
- Applied to all four sides of each tile
- Default: -1 (triggers prompt)

#### FR2.3: Spacing (`space=x,y`)
- Empty space added between tiles in pixels
- Required parameter (prompted if not provided)
- Applied after padding/extrusion
- Default: -1 (triggers prompt)

#### FR2.4: Diagonal Processing Mode (`diag=n`)
- Controls how diagonal corner pixels are filled
- Optional parameter
- Default: 3
- Modes:
  - 0: No diagonal fill
  - 1: Use first pixel (top-left corner)
  - 2: Average all four corner pixels
  - 3: Extrude each corner diagonally (recommended)

### FR3: Interactive Prompts
When required parameters are missing, application must:
- Display descriptive prompt
- Accept user input
- Validate input as integer
- Continue processing

### FR4: File Operations

#### FR4.1: Input File Validation
- Check file existence before processing
- Display error and exit if file not found

#### FR4.2: Output File Handling
- Auto-generate output filename if not specified: `<input>_extruded.<ext>`
- Prompt for overwrite confirmation if output exists
- Accept 'y' (case-insensitive) to overwrite
- Abort operation on any other input

#### FR4.3: Supported Image Formats
Must support common formats:
- PNG (recommended for transparency)
- BMP
- JPG/JPEG
- GIF
- TIFF
- Any format supported by System.Drawing.Bitmap

### FR5: Help System
- Display usage when no arguments provided
- Support `-h` and `-help` flags
- Show version number
- Display GitHub repository link
- List all parameters with descriptions
- Show diagonal mode options

### FR6: Edge Pixel Extrusion Algorithm

#### FR6.1: Corner Handling (Diagonal Fill)
Based on `diagProcessLevel`:
- **Mode 0**: No corner processing
- **Mode 1**: Fill entire corner area with top-left pixel
- **Mode 2**: Fill entire corner area with averaged ARGB values of all four corners
- **Mode 3**: Divide each corner into quadrants, fill each with respective corner pixel

#### FR6.2: Edge Extrusion
- Extract strips from tile edges
- Mirror/reverse the strips (bidirectional)
- Apply to padding area with negative width/height to flip

### FR7: Output Image Calculation
**Output dimensions**:
```
outputWidth = tilesX * (tileWidth + xPad * 2 + xSpace * 2)
outputHeight = tilesY * (tileHeight + yPad * 2 + ySpace * 2)
```

Where:
- `tilesX = inputWidth / tileWidth`
- `tilesY = inputHeight / tileHeight`

### FR8: DPI Preservation
- Read input image DPI (horizontal and vertical resolution)
- Apply same DPI to output image
- Prevent dimension corruption between 72dpi, 96dpi, etc.

## Non-Functional Requirements

### NFR1: Performance
- Process images efficiently using GDI+ Graphics API
- No specific performance targets documented
- Suitable for batch processing

### NFR2: Usability
- Clear error messages
- Intuitive parameter names
- Color-coded console output for help text
- Progress indication ("Processing...")

### NFR3: Compatibility
- Target: .NET Framework 4.0+
- Platform: Windows (due to System.Drawing)
- Architecture: AnyCPU (32-bit or 64-bit)

### NFR4: Maintainability
- Single-module architecture
- Clear function separation
- Inline comments for complex operations

### NFR5: Reliability
- Validate input file existence
- Prevent accidental file overwrites
- Handle missing parameters gracefully
- Try-catch blocks for argument parsing

## Input Validation Rules

1. **File Existence**: Input file must exist
2. **Tile Size**: Must be > 0, integer values
3. **Padding**: Must be >= 0, integer values
4. **Spacing**: Must be >= 0, integer values
5. **Diagonal Mode**: Must be 0-3, integer value
6. **Parameter Format**: `name=value` or `name=x,y`
7. **Single Value Shorthand**: `size=16` expands to `size=16,16`

## Edge Cases

1. **Pad > Tile Size**: Known issue - produces incorrect results
2. **Zero Padding**: Valid - no extrusion applied
3. **Zero Spacing**: Valid - tiles adjacent but extruded
4. **Non-square Tiles**: Fully supported with separate x,y values
5. **Partial Tiles**: Input dimensions must be exact multiples of tile size
6. **Missing Output Extension**: Uses input file extension
