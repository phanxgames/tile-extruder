# Migration Guide

## Overview

This document provides guidance for migrating TileExtruder from Visual Basic .NET to another programming language. It covers language-agnostic requirements, platform considerations, and recommendations for target languages.

## Migration Objectives

### Primary Goals
1. **Performance Improvement**: Target language should execute faster than VB.NET
2. **Cross-Platform Support**: Remove Windows-only dependency (System.Drawing/GDI+)
3. **Modern Tooling**: Leverage contemporary development ecosystems
4. **Maintainability**: Improve code structure and testability

### Must-Preserve Features
- All command-line arguments and behavior
- Exact same extrusion algorithms
- Output file compatibility (pixel-perfect matching)
- Interactive prompt mode
- Help system
- Overwrite confirmation

### Nice-to-Have Improvements
- Multi-threading support
- Progress bars for large files
- GPU acceleration option
- Better error messages
- Input validation (pad > tile size)
- Preserve original bit depth option
- Batch processing mode

## Language-Agnostic Requirements

### Core Functionality

#### 1. Command-Line Parsing
**Requirements**:
- Parse arguments in form `name=value` or `name=x,y`
- Support positional arguments (input file, output file)
- Detect flags (`-h`, `-help`)
- Handle missing arguments gracefully

**Considerations**:
- Use established CLI parsing library in target language
- Support both long (`--help`) and short (`-h`) flags
- Add validation for parameter ranges

#### 2. Image I/O
**Requirements**:
- Load common image formats: PNG, BMP, JPG, GIF, TIFF
- Preserve image metadata (DPI/resolution)
- Save in same format as input
- Handle transparency/alpha channel

**Recommended Libraries by Language**:
- **Rust**: `image` crate
- **C/C++**: `stb_image`, `libpng`, `libjpeg`
- **Python**: `PIL`/`Pillow`
- **Go**: `image` package + format-specific decoders
- **JavaScript/TypeScript**: `sharp`, `jimp`
- **C#**: `ImageSharp` (cross-platform, not GDI+)

#### 3. Pixel Manipulation
**Requirements**:
- Read pixel values (RGBA)
- Write pixel values
- Fill rectangular regions
- Copy/transform image regions
- Mirror/flip operations

**Implementation Options**:
1. **High-level**: Use library's drawing API
2. **Mid-level**: Direct pixel buffer manipulation
3. **Low-level**: Raw byte array operations (fastest)

#### 4. Graphics Operations
**Key Operations Needed**:
- `FillRectangle(color, x, y, width, height)`
- `CopyRegion(src, srcRect, dest, destRect)`
- `MirrorRegion(src, srcRect, dest, destRect, flipH, flipV)`

**Mirroring Implementation**:
```pseudo
function mirrorHorizontal(src, srcRect, dest, destX, destY):
    for y in 0 to srcRect.height:
        for x in 0 to srcRect.width:
            srcPixel = src.getPixel(srcRect.x + x, srcRect.y + y)
            dest.setPixel(destX + srcRect.width - 1 - x, destY + y, srcPixel)
```

### Algorithm Implementation

#### Diagonal Fill Modes

**Mode 0: No Fill**
```pseudo
// Skip diagonal processing
```

**Mode 1: First Pixel**
```pseudo
color = src.getPixel(tileX, tileY)
dest.fillRect(color, destX - padX, destY - padY,
              tileW + 2*padX, tileH + 2*padY)
```

**Mode 2: Average Corners**
```pseudo
tl = src.getPixel(tileX, tileY)
tr = src.getPixel(tileX + tileW - 1, tileY)
bl = src.getPixel(tileX, tileY + tileH - 1)
br = src.getPixel(tileX + tileW - 1, tileY + tileH - 1)

avgR = (tl.r + tr.r + bl.r + br.r) / 4
avgG = (tl.g + tr.g + bl.g + br.g) / 4
avgB = (tl.b + tr.b + bl.b + br.b) / 4
avgA = (tl.a + tr.a + bl.a + br.a) / 4

avgColor = Color(avgR, avgG, avgB, avgA)
dest.fillRect(avgColor, destX - padX, destY - padY,
              tileW + 2*padX, tileH + 2*padY)
```

**Mode 3: Extrude Corners**
```pseudo
halfW = tileW / 2
halfH = tileH / 2

// Top-left
tl = src.getPixel(tileX, tileY)
dest.fillRect(tl, destX - padX, destY - padY,
              halfW + 2*padX, halfH + 2*padY)

// Top-right
tr = src.getPixel(tileX + tileW - 1, tileY)
dest.fillRect(tr, destX + halfW - padX, destY - padY,
              halfW + 2*padX, halfH + 2*padY)

// Bottom-left
bl = src.getPixel(tileX, tileY + tileH - 1)
dest.fillRect(bl, destX - padX, destY + halfH - padY,
              halfW + 2*padX, halfH + 2*padY)

// Bottom-right
br = src.getPixel(tileX + tileW - 1, tileY + tileH - 1)
dest.fillRect(br, destX + halfW - padX, destY + halfH - padY,
              halfW + 2*padX, halfH + 2*padY)
```

#### Edge Extrusion

**Horizontal Edges**:
```pseudo
if padX > 0:
    // Left edge
    for y in 0 to tileH - 1:
        for x in 0 to padX - 1:
            pixel = src.getPixel(tileX + x, tileY + y)
            // Mirror: rightmost source becomes leftmost dest
            dest.setPixel(destX - padX + (padX - 1 - x), destY + y, pixel)

    // Right edge
    for y in 0 to tileH - 1:
        for x in 0 to padX - 1:
            pixel = src.getPixel(tileX + tileW - padX + x, tileY + y)
            // Mirror: leftmost source becomes rightmost dest
            dest.setPixel(destX + tileW + padX - 1 - x, destY + y, pixel)
```

**Vertical Edges**:
```pseudo
if padY > 0:
    // Top edge
    for y in 0 to padY - 1:
        for x in 0 to tileW - 1:
            pixel = src.getPixel(tileX + x, tileY + y)
            // Mirror: bottommost source becomes topmost dest
            dest.setPixel(destX + x, destY - padY + (padY - 1 - y), pixel)

    // Bottom edge
    for y in 0 to padY - 1:
        for x in 0 to tileW - 1:
            pixel = src.getPixel(tileX + x, tileY + tileH - padY + y)
            // Mirror: topmost source becomes bottommost dest
            dest.setPixel(destX + x, destY + tileH + padY - 1 - y, pixel)
```

## Target Language Recommendations

### Option 1: Rust (Highly Recommended)

**Pros**:
- Excellent performance (comparable to C++)
- Memory safety without garbage collection
- Cross-platform
- Great CLI ecosystem (`clap` crate)
- Excellent image library (`image` crate)
- Package manager (Cargo)
- Easy distribution (single binary)

**Cons**:
- Steeper learning curve
- Longer compile times

**Estimated Performance**: 5-10x faster than VB.NET

**Key Libraries**:
```toml
[dependencies]
clap = "4.0"           # CLI argument parsing
image = "0.24"         # Image processing
anyhow = "1.0"         # Error handling
indicatif = "0.17"     # Progress bars (optional)
rayon = "1.7"          # Parallelism (optional)
```

**Sample Code Structure**:
```rust
use image::{DynamicImage, GenericImage, GenericImageView, RgbaImage};

struct Config {
    input: String,
    output: String,
    tile_width: u32,
    tile_height: u32,
    pad_x: u32,
    pad_y: u32,
    space_x: u32,
    space_y: u32,
    diag_mode: u8,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = parse_args()?;
    process_image(&config)?;
    Ok(())
}

fn process_image(config: &Config) -> Result<(), Box<dyn std::error::Error>> {
    let img = image::open(&config.input)?;
    let output = extrude_tiles(&img, config);
    output.save(&config.output)?;
    Ok(())
}
```

### Option 2: C/C++

**Pros**:
- Maximum performance
- Full control over memory
- Mature image libraries
- Cross-platform

**Cons**:
- Manual memory management
- More complex build process
- Potential for bugs (buffer overflows, etc.)

**Estimated Performance**: 8-15x faster than VB.NET

**Key Libraries**:
- `stb_image.h` / `stb_image_write.h` (single-header, easy)
- `libpng`, `libjpeg` (feature-rich)
- CLI11 or cxxopts (argument parsing)

### Option 3: Go

**Pros**:
- Simple, clean syntax
- Fast compilation
- Cross-compilation support
- Built-in concurrency
- Single binary output
- Good standard library

**Cons**:
- Image library less mature than Rust
- Garbage collection (slight performance overhead)
- Verbose error handling

**Estimated Performance**: 3-5x faster than VB.NET

**Key Libraries**:
```go
import (
    "image"
    "image/png"
    "github.com/spf13/cobra"  // CLI
)
```

### Option 4: Python

**Pros**:
- Easiest to write
- Excellent libraries (Pillow)
- Great for prototyping
- Easy to maintain

**Cons**:
- Slowest option (without optimizations)
- Requires Python runtime
- Not ideal for performance-critical tasks

**Estimated Performance**: 2-3x slower than VB.NET (but can be optimized with NumPy)

**Use Case**: Prototyping or when performance isn't critical

**Optimization**: Use NumPy for pixel operations
```python
import numpy as np
from PIL import Image

# Direct pixel array manipulation
pixels = np.array(img)
# ... operations on numpy array ...
output = Image.fromarray(pixels)
```

### Option 5: C# with ImageSharp

**Pros**:
- Similar to VB.NET (easy migration)
- Cross-platform (.NET Core/.NET 5+)
- ImageSharp is fast and modern
- Familiar ecosystem

**Cons**:
- Still requires .NET runtime
- Performance similar to VB.NET (but better)
- Larger distribution size

**Estimated Performance**: 1.5-2x faster than VB.NET

**Key Libraries**:
```xml
<PackageReference Include="SixLabors.ImageSharp" Version="3.0" />
<PackageReference Include="CommandLineParser" Version="2.9" />
```

**Use Case**: Quickest migration path with cross-platform support

### Recommendation Summary

| Language | Performance | Development Speed | Distribution | Cross-Platform |
|----------|-------------|-------------------|--------------|----------------|
| Rust     | ⭐⭐⭐⭐⭐ | ⭐⭐⭐           | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⭐⭐      |
| C/C++    | ⭐⭐⭐⭐⭐ | ⭐⭐              | ⭐⭐⭐        | ⭐⭐⭐⭐        |
| Go       | ⭐⭐⭐⭐   | ⭐⭐⭐⭐          | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⭐⭐      |
| C#       | ⭐⭐⭐     | ⭐⭐⭐⭐⭐        | ⭐⭐⭐        | ⭐⭐⭐⭐        |
| Python   | ⭐⭐       | ⭐⭐⭐⭐⭐        | ⭐⭐          | ⭐⭐⭐⭐⭐      |

**Best Choice**: **Rust** - Optimal balance of performance, safety, and modern tooling

## Migration Checklist

### Phase 1: Setup
- [ ] Choose target language
- [ ] Set up development environment
- [ ] Select image processing library
- [ ] Select CLI parsing library
- [ ] Create project structure

### Phase 2: Core Implementation
- [ ] Implement argument parsing
- [ ] Add input validation
- [ ] Implement help system
- [ ] Load input image
- [ ] Calculate output dimensions
- [ ] Create output image
- [ ] Implement diagonal fill (all modes)
- [ ] Implement tile copying
- [ ] Implement horizontal extrusion
- [ ] Implement vertical extrusion
- [ ] Save output image
- [ ] Preserve DPI/resolution

### Phase 3: User Experience
- [ ] Implement interactive prompts
- [ ] Add file overwrite confirmation
- [ ] Display processing status
- [ ] Show output file size
- [ ] Color-coded console output (optional)

### Phase 4: Testing
- [ ] Test with various tile sizes
- [ ] Test all diagonal modes
- [ ] Test edge cases (small/large padding)
- [ ] Test different image formats
- [ ] Test transparency handling
- [ ] Verify pixel-perfect output vs VB.NET version
- [ ] Performance benchmarking

### Phase 5: Enhancements
- [ ] Add progress bar for large images
- [ ] Implement multi-threading
- [ ] Add validation (pad > tile size warning)
- [ ] Support batch processing
- [ ] Add configuration file support
- [ ] Improve error messages

### Phase 6: Distribution
- [ ] Build for multiple platforms (Windows, Linux, macOS)
- [ ] Create release packages
- [ ] Write updated documentation
- [ ] Create migration guide for users

## Backward Compatibility

### Command-Line Interface
Maintain 100% compatibility:
```bash
# Old VB.NET version
TileExtruder.exe input.png size=16 pad=2 space=1 output.png

# New version (should work identically)
tileextruder input.png size=16 pad=2 space=1 output.png
```

### Output Files
- Pixel-for-pixel identical output
- Same file formats supported
- Same DPI preservation
- Same default filenames

### Breaking Changes to Consider
**Optional Improvements** (document clearly):
- Add long-form flags: `--size`, `--padding`, `--spacing`, `--diagonal`
- Change default diagonal mode (currently 3)
- Add JSON/YAML config file support
- Change output naming convention

**Recommendation**: Keep old syntax, add new syntax as aliases

## Performance Optimization Strategies

### 1. Parallel Processing
```pseudo
// Process tiles in parallel
parallel_for tile in tiles:
    process_tile(tile)
```

**Speedup**: 4-8x on multi-core CPUs

### 2. Direct Pixel Buffer Access
```pseudo
// Instead of getPixel/setPixel
inputBuffer = image.getRawPixels()
outputBuffer = new byte[outputSize]

for i in range(pixelCount):
    outputBuffer[i] = transform(inputBuffer[i])

outputImage.setRawPixels(outputBuffer)
```

**Speedup**: 2-3x vs high-level API

### 3. SIMD Operations
```pseudo
// Process 4 pixels at once using SIMD
use simd instructions to copy/transform pixel data
```

**Speedup**: 2-4x additional

### 4. GPU Acceleration (Advanced)
- Implement as compute shader
- Use OpenGL, Vulkan, or CUDA
- Massive speedup for very large images

**Complexity**: High
**Speedup**: 10-100x for large images

## Testing Strategy

### Unit Tests
```pseudo
test_parse_args_single_value():
    args = parse("size=16")
    assert args.tile_width == 16
    assert args.tile_height == 16

test_parse_args_two_values():
    args = parse("size=16,32")
    assert args.tile_width == 16
    assert args.tile_height == 32

test_diagonal_mode_0():
    # Create 4x4 test image
    output = process(input, diag=0)
    # Verify corners are untouched

test_diagonal_mode_3():
    output = process(input, diag=3)
    # Verify each quadrant has correct corner color
```

### Integration Tests
```pseudo
test_full_pipeline():
    # Use reference input image
    output = run_extruder("test.png", size=16, pad=2, space=1)

    # Compare with known-good output
    assert output == reference_output

test_interactive_mode():
    # Simulate user input
    mock_stdin("16\n16\n2\n2\n1\n1\n")
    output = run_extruder("test.png")
    # Verify result
```

### Visual Regression Tests
- Process same inputs as VB.NET version
- Compare outputs pixel-by-pixel
- Ensure 100% identical results

## Documentation Updates

### Update These Sections
1. **README.md**: Installation instructions for new language
2. **Build instructions**: How to compile from source
3. **Platform support**: Emphasize cross-platform capability
4. **Performance**: Updated benchmarks
5. **Dependencies**: New library requirements

### New Documentation
1. **CONTRIBUTING.md**: Guidelines for contributors
2. **CHANGELOG.md**: Version history
3. **MIGRATION.md**: Guide for users switching from VB.NET version

## Distribution Strategy

### Single Binary Distribution
**Rust/Go**: Compile to static binary
```bash
# Rust
cargo build --release --target x86_64-pc-windows-gnu
cargo build --release --target x86_64-unknown-linux-gnu
cargo build --release --target x86_64-apple-darwin

# Go
GOOS=windows GOARCH=amd64 go build
GOOS=linux GOARCH=amd64 go build
GOOS=darwin GOARCH=amd64 go build
```

### Package Managers
- **Rust**: Publish to crates.io
- **Go**: Go modules
- **Python**: PyPI (pip install)
- **C#**: NuGet

### GitHub Releases
Attach binaries for:
- Windows x64
- Linux x64
- macOS x64 (Intel)
- macOS ARM64 (Apple Silicon)

## Migration Effort Estimate

| Language | Estimated Time | Difficulty | Testing Time |
|----------|---------------|------------|--------------|
| C# (ImageSharp) | 1-2 weeks | Easy | 1 week |
| Rust | 2-4 weeks | Medium | 1-2 weeks |
| Go | 2-3 weeks | Medium | 1 week |
| C++ | 3-5 weeks | Hard | 2 weeks |
| Python | 1 week | Easy | 1 week |

**Assumption**: Developer familiar with target language
