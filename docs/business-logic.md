# Business Logic and Algorithms

## Core Processing Flow

### Main Execution Flow (`Main()`)
```
1. Check if arguments provided
   └─> NO: Display usage and exit
   └─> YES: Continue

2. Check for help flag (-h or -help)
   └─> YES: Display help and exit
   └─> NO: Continue

3. Validate input file exists
   └─> NO: Display error and exit
   └─> YES: Continue

4. Parse input/output filenames
   - First argument = input file
   - Last argument = output file (if doesn't contain '=')
   - If no output specified: generate as "<input>_extruded<ext>"

5. Parse command-line arguments
   - Extract size, pad, space, diag parameters
   - Handle errors in parameter parsing

6. Prompt for missing required parameters
   - Tile size (if not provided)
   - Padding (if not provided)
   - Spacing (if not provided)

7. Execute image processing (Chooch())

8. Display output file size
```

### Argument Parsing (`ParseArg()`)

**Purpose**: Convert string arguments into structured data

**Input Format**: `name=value` or `name=x,y`

**Algorithm**:
```
1. Split argument by '=' and ','
2. Count resulting parts:

   Case: 3+ parts (e.g., "size=16,32")
      → Return [name, x, y]

   Case: 2 parts (e.g., "size=16")
      → Return [name, value, value]  // Duplicate value for y

   Case: 1 part (e.g., "inputfile.png")
      → Return [value, "0", "0"]
```

**Special Cases**:
- Single value duplicated for both x and y dimensions
- Allows shorthand: `pad=2` becomes `pad=2,2`

## Core Image Processing Algorithm (`Chooch()`)

### Phase 1: Initialization

```
1. Load input bitmap
2. Calculate tile grid dimensions:
   tilesX = inputWidth / tileWidth
   tilesY = inputHeight / tileHeight

3. Calculate output dimensions:
   outputWidth = tilesX * (tileWidth + xPad*2 + xSpace*2)
   outputHeight = tilesY * (tileHeight + yPad*2 + ySpace*2)

4. Create output bitmap with calculated dimensions
5. Preserve DPI from input to output
6. Create Graphics context from output bitmap
```

### Phase 2: Tile Processing Loop

**For each tile in grid (row-major order)**:

```
FOR y = 0 TO tilesY-1
  FOR x = 0 TO tilesX-1

    // Calculate coordinates
    srcRect = Rectangle(x * tileW, y * tileH, tileW, tileH)
    destX = x * (tileW + xSpace*2 + xPad*2) + xSpace + xPad
    destY = y * (tileH + ySpace*2 + yPad*2) + ySpace + yPad

    // Process this tile
    1. Apply diagonal corner fill (if enabled)
    2. Copy source tile to destination
    3. Extrude horizontal edges (if xPad > 0)
    4. Extrude vertical edges (if yPad > 0)

  NEXT x
NEXT y
```

### Phase 3: Diagonal Corner Fill

**Only executes if**: `(xPad > 0 OR yPad > 0) AND diagProcessLevel > 0`

#### Mode 3: Extrude Corners (Default, Recommended)

Divides each tile into quadrants and fills corner areas:

```
// Calculate half-tile dimensions
halfW = tileW / 2
halfH = tileH / 2

// Top-Left Corner
color = GetPixel(srcRect.X, srcRect.Y)
FillRectangle(color, destX - xPad, destY - yPad, halfW + xPad*2, halfH + yPad*2)

// Top-Right Corner
color = GetPixel(srcRect.X + tileW - 1, srcRect.Y)
FillRectangle(color, destX + halfW - xPad, destY - yPad, halfW + xPad*2, halfH + yPad*2)

// Bottom-Left Corner
color = GetPixel(srcRect.X, srcRect.Y + tileH - 1)
FillRectangle(color, destX - xPad, destY + halfH - yPad, halfW + xPad*2, halfH + yPad*2)

// Bottom-Right Corner
color = GetPixel(srcRect.X + tileW - 1, srcRect.Y + tileH - 1)
FillRectangle(color, destX + halfW - xPad, destY + halfH - yPad, halfW + xPad*2, halfH + yPad*2)
```

**Visual Representation**:
```
+--------+--------+
|   TL   |   TR   |  Each corner fills its quadrant
|  color |  color |  plus padding area
+--------+--------+
|   BL   |   BR   |
|  color |  color |
+--------+--------+
```

#### Mode 2: Average Corner Pixels

```
1. Sample all four corner pixels:
   px1 = GetPixel(topLeft)
   px2 = GetPixel(topRight)
   px3 = GetPixel(bottomLeft)
   px4 = GetPixel(bottomRight)

2. Average each channel:
   avgA = (px1.A + px2.A + px3.A + px4.A) / 4
   avgR = (px1.R + px2.R + px3.R + px4.R) / 4
   avgG = (px1.G + px2.G + px3.G + px4.G) / 4
   avgB = (px1.B + px2.B + px3.B + px4.B) / 4

3. Fill entire corner area with averaged color:
   FillRectangle(averagedColor, destX - xPad, destY - yPad,
                 tileW + xPad*2, tileH + yPad*2)
```

#### Mode 1: First Pixel

```
1. Get top-left corner pixel:
   color = GetPixel(srcRect.X, srcRect.Y)

2. Fill entire corner area:
   FillRectangle(color, destX - xPad, destY - yPad,
                 tileW + xPad*2, tileH + yPad*2)
```

#### Mode 0: No Fill
Skip corner processing entirely.

### Phase 4: Copy Source Tile

```
DrawImage(sourceBitmap, destX, destY, srcRect, Pixel)
```

This places the original tile in the center of the padded area.

### Phase 5: Horizontal Edge Extrusion

**Only if**: `xPad > 0`

#### Left Edge Extrusion:
```
1. Define strip from left edge of source tile:
   strip = Rectangle(srcRect.X, srcRect.Y, xPad, tileH)

2. Draw with NEGATIVE width to flip horizontally:
   DrawImage(source, Rectangle(destX, destY, -xPad, tileH), strip)
```

**Effect**: Takes the leftmost `xPad` pixels and mirrors them to the left.

#### Right Edge Extrusion:
```
1. Define strip from right edge of source tile:
   strip = Rectangle(srcRect.X + tileW - xPad, srcRect.Y, xPad, tileH)

2. Draw with NEGATIVE width to flip horizontally:
   DrawImage(source, Rectangle(destX + tileW + xPad, destY, -xPad, tileH), strip)
```

**Effect**: Takes the rightmost `xPad` pixels and mirrors them to the right.

### Phase 6: Vertical Edge Extrusion

**Only if**: `yPad > 0`

#### Top Edge Extrusion:
```
1. Define strip from top edge of source tile:
   strip = Rectangle(srcRect.X, srcRect.Y, tileW, yPad)

2. Draw with NEGATIVE height to flip vertically:
   DrawImage(source, Rectangle(destX, destY, tileW, -yPad), strip)
```

**Effect**: Takes the topmost `yPad` pixels and mirrors them upward.

#### Bottom Edge Extrusion:
```
1. Define strip from bottom edge of source tile:
   strip = Rectangle(srcRect.X, srcRect.Y + tileH - yPad, tileW, yPad)

2. Draw with NEGATIVE height to flip vertically:
   DrawImage(source, Rectangle(destX, destY + tileH + yPad, tileW, -yPad), strip)
```

**Effect**: Takes the bottommost `yPad` pixels and mirrors them downward.

### Phase 7: File Output

```
1. Check if output file exists:

   IF exists:
      Prompt user: "already exists. Overwrite? (y/n): "
      Read single key

      IF key = 'y' or 'Y':
         Delete existing file
         Save new bitmap
      ELSE:
         Display "Operation aborted."
         Exit without saving

   ELSE:
      Save bitmap directly
```

## Key Algorithms Explained

### Bidirectional (Bidi) Mirroring

The term "bidi" refers to the mirroring technique used for edge extrusion:

**Example - Left edge extrusion with pad=2**:
```
Source tile left edge:    Extruded result:
Pixels: [A B C D ...]     [B A | A B C D ...]
                           ----   -----------
                          Mirror  Original
```

**Implementation**: Using negative width/height in `DrawImage()` causes GDI+ to flip the image.

### Spacing vs Padding

**Padding** (`pad`):
- Extrudes pixels outward from tile edges
- Creates seamless transition
- Part of the tile's "extended" area

**Spacing** (`space`):
- Empty pixels between tiles
- Prevents tiles from bleeding into neighbors
- Applied outside the padding area

**Visual**:
```
[Space][Pad][TILE][Pad][Space][Space][Pad][TILE][Pad][Space]
 <---> <-->  <-->  <--> <----> <---> <-->  <-->  <--> <--->
  gap  extr  orig  extr   gap    gap  extr  orig  extr  gap
```

### Coordinate Calculation

**Destination X position**:
```
destX = tileIndex_X * (tileWidth + xSpace*2 + xPad*2) + xSpace + xPad
        |____________________________________________|   |___________|
                     Stride per tile                      Offset
```

**Breakdown**:
- `tileWidth`: Original tile size
- `xPad*2`: Padding on both left and right
- `xSpace*2`: Spacing on both left and right
- `+ xSpace + xPad`: Offset to skip left spacing and padding

## Mathematical Formulas

### Output Dimensions
```
outputWidth = ⌊inputWidth / tileWidth⌋ * (tileWidth + 2*xPad + 2*xSpace)
outputHeight = ⌊inputHeight / tileHeight⌋ * (tileHeight + 2*yPad + 2*ySpace)
```

### Tile Position Mapping
```
srcTileX = tileIndexX * tileWidth
srcTileY = tileIndexY * tileHeight

destTileX = tileIndexX * (tileWidth + 2*xPad + 2*xSpace) + xSpace + xPad
destTileY = tileIndexY * (tileHeight + 2*yPad + 2*ySpace) + ySpace + yPad
```

### Padding Area Dimensions
```
leftPadArea: width=xPad, height=tileHeight
rightPadArea: width=xPad, height=tileHeight
topPadArea: width=tileWidth, height=yPad
bottomPadArea: width=tileWidth, height=yPad
```

### Corner Area Dimensions (Mode 3)
```
cornerWidth = tileWidth/2 + 2*xPad
cornerHeight = tileHeight/2 + 2*yPad
```

## Processing Order and Layering

The order of operations is critical:

1. **Corner fill** (background layer)
2. **Source tile copy** (center, covers corner fill in tile area)
3. **Horizontal extrusion** (left/right edges, covers corner fill on sides)
4. **Vertical extrusion** (top/bottom edges, covers corner fill on top/bottom)

This layering ensures proper coverage with the correct pixel data.

## Error Handling

### Argument Parsing Errors
```
TRY
   Parse argument value
   Set parameter
CATCH Exception
   Display: "Error: Invalid parameter for argument '<name>'"
   Exit program
```

### File Not Found
```
IF NOT FileExists(input)
   Display: "Error: Input file does not exist."
   Exit program
```

### User Abort
```
// No error thrown, just message
Display: "Operation aborted."
// Program ends normally
```
