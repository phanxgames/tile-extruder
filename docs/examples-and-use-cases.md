# Examples and Use Cases

## Command-Line Usage Examples

### Example 1: Basic Usage with All Parameters
```bash
TileExtruder input.png size=16 pad=2 space=1 output.png
```

**What happens**:
- Input: `input.png`
- Tile size: 16x16 pixels
- Padding: 2 pixels on each side
- Spacing: 1 pixel between tiles
- Output: `output.png`

**Result**:
- Each 16x16 tile becomes 22x22 in output (16 + 2*2 pad + 2*1 space)

### Example 2: Non-Square Tiles
```bash
TileExtruder tileset.png size=32,16 pad=3,2 space=1,0
```

**What happens**:
- Tile size: 32 pixels wide, 16 pixels tall
- Padding: 3 pixels horizontally, 2 pixels vertically
- Spacing: 1 pixel horizontally, 0 pixels vertically
- Output: Auto-generated as `tileset_extruded.png`

### Example 3: No Spacing, Only Padding
```bash
TileExtruder tiles.png size=8 pad=1 space=0
```

**Use case**: Maximum extrusion without gaps between tiles

### Example 4: Interactive Mode
```bash
TileExtruder mysheet.png
```

**Console interaction**:
```
Tile X Size: 16
Tile Y Size: 16

X Padding: 2
Y Padding: 2

X Spacing: 1
Y Spacing: 1

Processing...
Wrote 524288 bytes.
```

### Example 5: Different Diagonal Modes
```bash
# Mode 0: No diagonal fill
TileExtruder input.png size=16 pad=2 space=0 diag=0

# Mode 1: First pixel
TileExtruder input.png size=16 pad=2 space=0 diag=1

# Mode 2: Averaged corners
TileExtruder input.png size=16 pad=2 space=0 diag=2

# Mode 3: Extrude corners (default)
TileExtruder input.png size=16 pad=2 space=0 diag=3
```

### Example 6: Help Display
```bash
TileExtruder -h
```

**Output**:
```
TileExtruder 1.0.0.0 by Nobuyuki.
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

## Visual Examples

### Visual Example 1: Padding Effect

**Input Tile (4x4)**:
```
A B C D
E F G H
I J K L
M N O P
```

**With pad=1**:
```
F E │ E F G H │ H G
B A │ A B C D │ D C
────┼─────────┼────
B A │ A B C D │ D C
F E │ E F G H │ H G
J I │ I J K L │ L K
N M │ M N O P │ P O
────┼─────────┼────
N M │ M N O P │ P O
J I │ I J K L │ L K
```

Legend:
- Center: Original tile
- Sides: Mirrored edges
- Corners: Depends on diagonal mode

### Visual Example 2: Spacing Effect

**Without spacing** (pad=1, space=0):
```
[Tile1][Tile2][Tile3]
```

**With spacing** (pad=1, space=1):
```
[Tile1] [Tile2] [Tile3]
        ↑
      1px gap
```

### Visual Example 3: Diagonal Mode Comparison

**4x4 Tile with pad=2**:

**Mode 0 (No fill)**:
```
? ? │ ? ? ? ? │ ? ?
? ? │ ? ? ? ? │ ? ?
────┼─────────┼────
? ? │ A B C D │ ? ?
? ? │ E F G H │ ? ?
? ? │ I J K L │ ? ?
? ? │ M N O P │ ? ?
────┼─────────┼────
? ? │ ? ? ? ? │ ? ?
? ? │ ? ? ? ? │ ? ?
```
(? = undefined/transparent)

**Mode 1 (First pixel 'A')**:
```
A A │ A A A A │ A A
A A │ A A A A │ A A
────┼─────────┼────
A A │ A B C D │ A A
A A │ E F G H │ A A
A A │ I J K L │ A A
A A │ M N O P │ A A
────┼─────────┼────
A A │ A A A A │ A A
A A │ A A A A │ A A
```

**Mode 3 (Extrude corners)**:
```
A A │ A A D D │ D D
A A │ A A D D │ D D
────┼─────────┼────
A A │ A B C D │ D D
A A │ E F G H │ D D
M M │ I J K L │ P P
M M │ M N O P │ P P
────┼─────────┼────
M M │ M M P P │ P P
M M │ M M P P │ P P
```

## Use Cases

### Use Case 1: 2D Game Tilemap

**Scenario**: Platformer game with 16x16 pixel tiles

**Problem**: Tiles show seams when camera moves or zooms due to bilinear filtering

**Solution**:
```bash
TileExtruder terrain.png size=16 pad=2 space=0 diag=3
```

**Integration**:
- Load `terrain_extruded.png` in game engine
- Configure tilemap:
  - Tile size: 16x16
  - Margin: 2
  - Spacing: 2
- Seams eliminated!

### Use Case 2: Sprite Atlas for Mobile Game

**Scenario**: Mobile game needs optimized sprite atlas

**Problem**: Texture bleeding between sprites causing visual artifacts

**Solution**:
```bash
TileExtruder sprites.png size=32,32 pad=1 space=2 diag=3
```

**Benefits**:
- 1px padding prevents bleeding
- 2px spacing provides safety margin
- Corner extrusion handles rotation

### Use Case 3: Tiled Map Editor Integration

**Scenario**: Using Tiled (mapeditor.org) for level design

**Problem**: Tiled's renderer shows seams with non-extruded tilesets

**Solution**:
```bash
TileExtruder tileset.png size=16 pad=1 space=0
```

**Tiled Configuration**:
- Margin: 1
- Spacing: 2 (pad*2)
- Tile size: 16x16

**TMX Format**:
```xml
<tileset firstgid="1" name="terrain" tilewidth="16" tileheight="16"
         spacing="2" margin="1">
  <image source="tileset_extruded.png" width="..." height="..."/>
</tileset>
```

### Use Case 4: Pixel Art Upscaling Pipeline

**Scenario**: Preparing pixel art for high-DPI displays

**Workflow**:
1. Extrude tileset:
   ```bash
   TileExtruder original.png size=8 pad=1 space=0
   ```

2. Upscale with nearest-neighbor:
   ```bash
   # Using ImageMagick or similar
   convert original_extruded.png -scale 400% upscaled.png
   ```

**Result**: Clean upscaling without seams

### Use Case 5: Texture Packing for WebGL

**Scenario**: Creating texture atlas for WebGL game

**Problem**: UV coordinate precision issues cause bleeding

**Solution**:
```bash
TileExtruder atlas.png size=64,64 pad=2 space=4 diag=3
```

**UV Mapping**:
- Account for padding in UV calculations
- Use spacing as safety buffer for mipmaps

### Use Case 6: Batch Processing Multiple Tilesets

**Scenario**: Game with 50+ tilesets needing extrusion

**Solution** (Windows batch script):
```batch
@echo off
for %%f in (*.png) do (
    TileExtruder "%%f" size=16 pad=2 space=1
)
```

**Solution** (Linux/Mac bash script):
```bash
#!/bin/bash
for file in *.png; do
    ./TileExtruder "$file" size=16 pad=2 space=1
done
```

## Real-World Scenarios

### Scenario 1: Godot Engine Integration

**Problem**: Godot's 2D renderer shows seams with TileMap

**Steps**:
1. Extrude tileset:
   ```bash
   TileExtruder tileset.png size=16 pad=1 space=0
   ```

2. Import in Godot
3. Configure TileSet resource:
   - Separation: 2
   - Margin: 1

4. Use in TileMap node

**Reference**: Godot expects spacing = 2*padding, margin = padding

### Scenario 2: Unity Integration

**Problem**: Unity's Tilemap system shows artifacts on mobile

**Steps**:
1. Extrude:
   ```bash
   TileExtruder unity_tiles.png size=32 pad=2 space=0
   ```

2. Import sprite sheet
3. Slice with:
   - Cell size: 32x32
   - Offset: 2, 2
   - Padding: 4, 4

### Scenario 3: Phaser.js Web Game

**JavaScript setup**:
```javascript
// After extruding with pad=2, space=0
this.load.spritesheet('tiles', 'tileset_extruded.png', {
    frameWidth: 16,
    frameHeight: 16,
    margin: 2,
    spacing: 4
});
```

### Scenario 4: RPG Maker MV/MZ

**Preparation**:
```bash
TileExtruder rpgmaker_tileset.png size=48 pad=1 space=0
```

**Note**: May require custom plugin to support margin/spacing

## Performance Examples

### Small Tileset
```
Input:  256x256 pixels, 16x16 tiles (16x16 grid = 256 tiles)
Params: pad=2, space=1
Output: 352x352 pixels
Time:   < 0.5 seconds
```

### Medium Tileset
```
Input:  1024x1024 pixels, 32x32 tiles (32x32 grid = 1024 tiles)
Params: pad=2, space=1
Output: 1408x1408 pixels
Time:   2-3 seconds
```

### Large Tileset
```
Input:  4096x4096 pixels, 16x16 tiles (256x256 grid = 65,536 tiles)
Params: pad=2, space=1
Output: 5632x5632 pixels
Time:   15-25 seconds
```

## Edge Cases and Solutions

### Edge Case 1: Transparent Tiles

**Problem**: Tiles with transparency may show background color in padding

**Solution**: Use diagonal mode 3 (extrude corners) to preserve transparency

**Example**:
```bash
TileExtruder sprites.png size=32 pad=2 space=0 diag=3
```

### Edge Case 2: Gradient Edges

**Problem**: Tiles with gradients may show discontinuity

**Solution**: Use smaller padding values (pad=1) for smoother transition

### Edge Case 3: Tiles with Borders

**Problem**: Tiles with borders may double the border in padding

**Recommendation**: Design tiles without borders, or use pad=0

### Edge Case 4: Non-Divisible Dimensions

**Problem**: Input 250x250 with size=16 results in partial tiles

**Current Behavior**: Only processes complete tiles (15x15 grid)

**Solution**: Crop input to exact multiple (240x240) before processing

### Edge Case 5: Very Large Padding

**Problem**: pad=10 on 8x8 tiles causes overlap and artifacts

**Known Issue**: No validation prevents this

**Workaround**: Keep pad < min(tileWidth, tileHeight)

## Common Workflows

### Workflow 1: From Photoshop to Game Engine

1. **Design** tiles in Photoshop (e.g., 16x16 grid)
2. **Export** as PNG
3. **Extrude**:
   ```bash
   TileExtruder exported.png size=16 pad=2 space=1
   ```
4. **Import** `exported_extruded.png` to game engine
5. **Configure** tilemap with margin=2, spacing=4

### Workflow 2: Asset Store to Production

1. **Purchase** tileset from asset store
2. **Verify** tile size (check documentation)
3. **Extrude** with appropriate parameters
4. **Test** in game to verify no seams
5. **Adjust** padding if needed

### Workflow 3: Procedural Tileset Generation

1. **Generate** tileset programmatically
2. **Save** as PNG
3. **Extrude** via command line
4. **Load** in application dynamically

## Integration Examples

### Python Script Integration
```python
import subprocess
import os

def extrude_tileset(input_file, tile_size, pad, space):
    output_file = input_file.replace('.png', '_extruded.png')

    cmd = [
        'TileExtruder.exe',
        input_file,
        f'size={tile_size}',
        f'pad={pad}',
        f'space={space}',
        output_file
    ]

    subprocess.run(cmd, check=True)
    return output_file

# Usage
extrude_tileset('tileset.png', 16, 2, 1)
```

### Node.js Build Script
```javascript
const { execSync } = require('child_process');

function extrudeTileset(input, size, pad, space) {
    const output = input.replace('.png', '_extruded.png');

    const cmd = `TileExtruder.exe ${input} size=${size} pad=${pad} space=${space} ${output}`;

    execSync(cmd);
    return output;
}

// Usage
extrudeTileset('tiles.png', 16, 2, 1);
```

### Makefile Integration
```makefile
TILESETS = $(wildcard assets/tilesets/*.png)
EXTRUDED = $(TILESETS:.png=_extruded.png)

all: $(EXTRUDED)

%_extruded.png: %.png
	TileExtruder.exe $< size=16 pad=2 space=1 $@

clean:
	rm -f assets/tilesets/*_extruded.png
```
