# Data Flow and State Management

## Application State Flow

```
┌─────────────┐
│   START     │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────┐
│  Parse Command Line Arguments       │
│  - input file path                  │
│  - output file path (optional)      │
│  - size=x,y                         │
│  - pad=x,y                          │
│  - space=x,y                        │
│  - diag=n                           │
└──────┬──────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────┐
│  Validate Required Parameters       │
│  - Check if size provided           │
│  - Check if pad provided            │
│  - Check if space provided          │
└──────┬──────────────────────────────┘
       │
       │ Missing params?
       ├─── YES ──┐
       │          ▼
       │   ┌──────────────────────┐
       │   │ Interactive Prompts  │
       │   │ - Prompt for size    │
       │   │ - Prompt for pad     │
       │   │ - Prompt for space   │
       │   └──────┬───────────────┘
       │          │
       ▼          │
       ◄──────────┘
       │
       ▼
┌─────────────────────────────────────┐
│  Initialize Image Processing        │
│  - Load input bitmap                │
│  - Calculate tile grid              │
│  - Calculate output dimensions      │
│  - Create output bitmap             │
│  - Create Graphics context          │
└──────┬──────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────┐
│  Process Each Tile (Nested Loop)    │
│  FOR y = 0 TO tilesY-1              │
│    FOR x = 0 TO tilesX-1            │
│      - Calculate coordinates        │
│      - Apply diagonal fill          │
│      - Copy source tile             │
│      - Extrude horizontal edges     │
│      - Extrude vertical edges       │
│    NEXT x                           │
│  NEXT y                             │
└──────┬──────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────┐
│  Output File                        │
│  - Check if file exists             │
│  - Prompt for overwrite (if needed) │
│  - Save bitmap                      │
│  - Display file size                │
└──────┬──────────────────────────────┘
       │
       ▼
┌──────────────┐
│     END      │
└──────────────┘
```

## Variable State Throughout Execution

### Initial State (Module Load)
```
input:  null
output: null
TileW:  0
TileH:  0
xPad:   -1
yPad:   0
xSpace: -1
ySpace: 0
diagProcessLevel: 3
```

### After Argument Parsing
```
input:  "input.png"
output: "output.png" OR auto-generated
TileW:  16 (if provided)
TileH:  16 (if provided)
xPad:   2 (if provided) OR -1
yPad:   2 (if provided) OR 0
xSpace: 1 (if provided) OR -1
ySpace: 1 (if provided) OR 0
diagProcessLevel: 0-3 (if provided) OR 3
```

### After Interactive Prompts (if needed)
```
All values >= 0 (all required values set)
```

### During Processing (Chooch)

**Local Variables Created**:
```
b:       Input Bitmap object
o:       Output Bitmap object
g:       Graphics context
xTiles:  Calculated tile count horizontally
yTiles:  Calculated tile count vertically
tw:      TileW / 2 (half-tile width)
th:      TileH / 2 (half-tile height)
```

**Per-Tile Variables** (loop iteration):
```
x, y:       Tile indices
srcRect:    Rectangle defining source tile region
destx:      Destination X coordinate
desty:      Destination Y coordinate
fill:       Color for diagonal fill
strip:      Rectangle for edge extrusion
```

## Data Transformations

### 1. Command-Line String → Structured Parameters

**Input**: `"size=16,32 pad=2 space=1,2"`

**Transformation** (`ParseArg`):
```
"size=16,32" → ["size", "16", "32"]
"pad=2"      → ["pad", "2", "2"]    // Duplicate value
"space=1,2"  → ["space", "1", "2"]
```

**Storage**:
```
TileW=16, TileH=32
xPad=2, yPad=2
xSpace=1, ySpace=2
```

### 2. Input Image → Bitmap Object

**File Path** → **Bitmap**:
```vb
input: "tileset.png" (String)
   ↓
   Bitmap constructor reads file
   ↓
b: Bitmap object in memory
   - Width: 256
   - Height: 256
   - PixelFormat: Format32bppArgb
   - HorizontalResolution: 96 DPI
   - VerticalResolution: 96 DPI
```

### 3. Bitmap Dimensions → Tile Grid

```
Input Bitmap: 256x256
Tile Size: 16x16

Calculation:
xTiles = 256 / 16 = 16
yTiles = 256 / 16 = 16

Result: 16x16 grid of tiles (256 total tiles)
```

### 4. Parameters → Output Dimensions

```
Input:
- xTiles: 16
- yTiles: 16
- TileW: 16, TileH: 16
- xPad: 2, yPad: 2
- xSpace: 1, ySpace: 1

Calculation per tile:
width_per_tile = 16 + (2*2) + (2*1) = 16 + 4 + 2 = 22
height_per_tile = 16 + (2*2) + (2*1) = 16 + 4 + 2 = 22

Output dimensions:
outputWidth = 16 * 22 = 352
outputHeight = 16 * 22 = 352
```

### 5. Tile Index → Source Rectangle

```
Tile at position (x=3, y=2):

srcRect.X = 3 * 16 = 48
srcRect.Y = 2 * 16 = 32
srcRect.Width = 16
srcRect.Height = 16

Result: Rectangle(48, 32, 16, 16)
```

### 6. Tile Index → Destination Coordinates

```
Tile at position (x=3, y=2):
Parameters: TileW=16, xPad=2, xSpace=1

destX = 3 * (16 + 2*2 + 2*1) + 1 + 2
      = 3 * 22 + 3
      = 66 + 3
      = 69

destY = 2 * (16 + 2*2 + 2*1) + 1 + 2
      = 2 * 22 + 3
      = 44 + 3
      = 47

Result: Destination position (69, 47)
```

### 7. Edge Pixels → Extruded Strips

**Left Edge Extrusion**:
```
Source:
strip = Rectangle(srcX, srcY, xPad=2, TileH=16)
      = First 2 columns of tile

Destination:
Rectangle(destX, destY, -2, 16)
Negative width flips horizontally

Visual:
Source strip: [A B]
Result:       [B A][A B][tile...]
              <-->  <--> Original
            Mirrored First
                    cols
```

### 8. Corner Pixels → Diagonal Fill (Mode 3)

**Top-Left Corner**:
```
Sample: GetPixel(srcX, srcY) = Color(R=255, G=0, B=0, A=255)

Fill region:
X: destX - xPad = destX - 2
Y: destY - yPad = destY - 2
Width: tileW/2 + 2*xPad = 8 + 4 = 12
Height: tileH/2 + 2*yPad = 8 + 4 = 12

Result: 12x12 red square in top-left corner area
```

## Memory Layout

### Input Bitmap Memory
```
For 256x256 image at 32bpp:
Size = 256 * 256 * 4 bytes = 262,144 bytes = 256 KB

Layout (row-major):
[Row 0: Pixel(0,0), Pixel(1,0), ..., Pixel(255,0)]
[Row 1: Pixel(0,1), Pixel(1,1), ..., Pixel(255,1)]
...
[Row 255: Pixel(0,255), Pixel(1,255), ..., Pixel(255,255)]

Each pixel: [B, G, R, A] (4 bytes)
```

### Output Bitmap Memory
```
For 352x352 output at 32bpp (from example above):
Size = 352 * 352 * 4 bytes = 495,616 bytes = 484 KB

Memory increase: +87.5%
```

## Graphics Pipeline

### GDI+ Operation Sequence

```
1. Graphics.FillRectangle()
   ├─ Create brush with color
   ├─ Fill rectangular region
   └─ Release brush

2. Graphics.DrawImage() [Normal]
   ├─ Define source rectangle
   ├─ Define destination rectangle
   ├─ Copy pixels (no transformation)
   └─ Apply to output bitmap

3. Graphics.DrawImage() [Mirrored]
   ├─ Define source rectangle
   ├─ Define destination with negative dimension
   ├─ GDI+ applies flip transformation
   ├─ Copy transformed pixels
   └─ Apply to output bitmap
```

### Rendering Order Per Tile

```
Layer 1 (Background):  Diagonal corner fill
Layer 2 (Center):      Source tile copy
Layer 3 (Sides):       Horizontal edge extrusion
Layer 4 (Top/Bottom):  Vertical edge extrusion

Visual result:
┌───────────────────────┐
│ Corner │ Top  │ Corner│ ← Layer 1, 4
├────────┼──────┼───────┤
│ Left   │ Tile │ Right │ ← Layer 2, 3
├────────┼──────┼───────┤
│ Corner │Bottom│ Corner│ ← Layer 1, 4
└───────────────────────┘
```

## File I/O Data Flow

### Read Path
```
File System → FileStream → GDI+ Decoder → Bitmap Object
                                    ↓
                            Pixel Data in Memory (32bpp)
```

### Write Path
```
Bitmap Object → GDI+ Encoder → FileStream → File System
                       ↓
                 Format-specific
                 (PNG, BMP, etc.)
```

### Format Conversion
```
Input: 8bpp indexed PNG
   ↓ GDI+ loads
Bitmap: 32bpp ARGB (in memory)
   ↓ GDI+ saves
Output: 32bpp PNG (file)

Note: Original bit depth not preserved
```

## Console I/O Flow

### Input Flow
```
User Keyboard → Console.ReadLine() → String → Int32.TryParse() → Integer
                                        ↓
                                  Validation
                                        ↓
                                  Assign to variable
```

### Output Flow
```
Variable/Constant → String.Format / Concatenation → Console.Write() → Terminal Display
                                                            ↓
                                                    Color formatting applied
```

## Resource Management

### Disposable Objects
```vb
Dim b As New Bitmap(input)        ' ← Created
Dim o As New Bitmap(...)          ' ← Created
Dim g As Graphics = Graphics.FromImage(o)  ' ← Created

' ... processing ...

g.Dispose()                        ' ← Manually disposed

' Note: Bitmaps b and o NOT explicitly disposed
' Rely on GC (garbage collector) - potential resource leak
```

**Recommendation**: Wrap in `Using` blocks for proper disposal:
```vb
Using b As New Bitmap(input)
    Using o As New Bitmap(...)
        Using g As Graphics = Graphics.FromImage(o)
            ' Process
        End Using
    End Using
End Using
```

## Error Propagation

### Exception Handling Flow
```
Try-Catch in argument parsing:
   ArgumentException → Caught → Display error → End program

No Try-Catch in Chooch():
   IOException → Uncaught → Program crash
   OutOfMemoryException → Uncaught → Program crash
   GDI+ exceptions → Uncaught → Program crash
```

**Risk**: File I/O and image operations can fail without graceful handling.

## State Mutation Timeline

```
Time →

T0: [Module loaded, defaults set]
T1: [Arguments parsed]
T2: [User prompts completed] ← All required state set
T3: [Bitmaps created] ← Immutable from here
T4: [Processing loop] ← Read-only access to state
T5: [File written]
T6: [Exit]
```

**Key Point**: After T2, all state is read-only. No mutation during processing.

## Concurrency Considerations

**Current Implementation**: Single-threaded, sequential processing

**Potential Parallelization**:
```
Independent tiles could be processed in parallel:

Thread 1: Process tiles (0,0), (2,0), (4,0), ...
Thread 2: Process tiles (1,0), (3,0), (5,0), ...
Thread 3: Process tiles (0,1), (2,1), (4,1), ...
...

Constraint: Graphics object is NOT thread-safe
Solution: Create separate Graphics context per thread
```

## Data Dependencies

```
Input File
   ↓
Bitmap 'b'
   ↓
   ├─→ Width, Height → Calculate xTiles, yTiles
   ├─→ HorizontalResolution, VerticalResolution → Set output DPI
   ├─→ GetPixel() → Sample edge and corner colors
   └─→ DrawImage source → Copy regions

Parameters (size, pad, space)
   ↓
Output Dimensions
   ↓
Bitmap 'o'
   ↓
Graphics 'g'
   ↓
Drawing Operations
   ↓
Output File
```

No circular dependencies. Clean unidirectional flow.
