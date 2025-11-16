# TileExtruder Documentation

## Overview

This documentation provides comprehensive information about the TileExtruder project for the purpose of migrating it to another programming language.

**Current Implementation**: Visual Basic .NET (VB.NET)
**Target**: Any high-performance language (Rust, C++, Go, etc.)

## Documentation Structure

### 1. [Project Overview](./project-overview.md)
High-level introduction to TileExtruder's purpose, technology stack, and use cases.

**Contents**:
- Problem statement (tileset seams)
- Solution approach (pixel extrusion)
- Current technology (VB.NET, .NET Framework 4.0)
- Known limitations
- Target audience

**Read this first** to understand what TileExtruder does and why it exists.

### 2. [Requirements Specification](./requirements.md)
Complete functional and non-functional requirements.

**Contents**:
- Functional requirements (FR1-FR8)
- Non-functional requirements (NFR1-NFR5)
- Input validation rules
- Edge cases
- Parameter specifications

**Use this** to ensure your implementation meets all requirements.

### 3. [Business Logic](./business-logic.md)
Detailed explanation of all algorithms and processing logic.

**Contents**:
- Main execution flow
- Argument parsing algorithm
- Core image processing (Chooch algorithm)
- Diagonal fill modes (0-3)
- Edge extrusion (bidirectional mirroring)
- Mathematical formulas
- Processing order and layering

**Use this** to understand exactly how the application works.

### 4. [Technical Specification](./technical-specification.md)
Implementation details, dependencies, and architecture.

**Contents**:
- Architecture diagram
- Technology stack
- Data structures
- GDI+ API usage
- Time/space complexity analysis
- Performance characteristics
- Security considerations

**Use this** for low-level implementation details.

### 5. [Data Flow](./data-flow.md)
How data moves through the application.

**Contents**:
- Application state flow diagram
- Variable state timeline
- Data transformations
- Memory layout
- Graphics pipeline
- Resource management

**Use this** to understand data dependencies and processing flow.

### 6. [Examples and Use Cases](./examples-and-use-cases.md)
Practical usage examples and integration patterns.

**Contents**:
- Command-line usage examples
- Visual examples of extrusion
- Real-world use cases (game engines)
- Integration examples (Python, Node.js)
- Batch processing scripts

**Use this** for testing and understanding expected behavior.

### 7. [API Specification](./api-specification.md)
Complete reference for command-line interface and functions.

**Contents**:
- CLI syntax specification
- Argument reference
- Function signatures
- Return codes
- Error conditions
- Output format specification

**Use this** as definitive reference for behavior.

### 8. [Migration Guide](./migration-guide.md)
Guidance for porting to another language.

**Contents**:
- Migration objectives
- Language-agnostic requirements
- Target language recommendations (Rust, C++, Go, Python, C#)
- Performance optimization strategies
- Testing strategy
- Migration checklist

**Use this** to plan and execute the migration.

## Quick Reference

### What TileExtruder Does

1. **Input**: Tileset image (e.g., 256×256 pixels with 16×16 tiles)
2. **Process**: Extrude edge pixels outward, add spacing
3. **Output**: Processed image with no rendering seams

### Core Algorithm

```
For each tile in tileset:
  1. Fill corner areas (diagonal mode)
  2. Copy original tile to center
  3. Mirror left/right edges horizontally
  4. Mirror top/bottom edges vertically
```

### Key Parameters

- **size**: Tile dimensions (e.g., `16` or `16,32`)
- **pad**: Extrusion amount (e.g., `2`)
- **space**: Gap between tiles (e.g., `1`)
- **diag**: Corner fill mode (0-3, default: 3)

### Example Usage

```bash
TileExtruder tileset.png size=16 pad=2 space=1 output.png
```

**Result**: 16×16 tiles become 22×22 in output (16 + 2×2 + 2×1)

## Migration Quick Start

### Recommended Approach

1. **Choose Language**: Rust (recommended for performance + safety)
2. **Select Libraries**:
   - Image I/O: `image` crate (Rust), `stb_image` (C++), etc.
   - CLI parsing: `clap` (Rust), `CLI11` (C++), etc.
3. **Implement Core Algorithm**: Follow pseudocode in Business Logic doc
4. **Test Against Reference**: Ensure pixel-perfect match with VB.NET version
5. **Optimize**: Add parallelization, SIMD, etc.

### Critical Implementation Details

#### 1. Bidirectional Mirroring
The "bidi" extrusion is achieved by **mirroring edge pixels**:

```
Source edge: [A B C]
Extruded:    [C B A | A B C]
             ------   ------
             Mirror   Original
```

**VB.NET Implementation**: DrawImage with negative width/height
**Your Implementation**: Manually reverse pixel order when copying

#### 2. Diagonal Mode 3 (Quadrant-Based)
Each corner of the tile fills its respective quadrant:

```
+--------+--------+
|   TL   |   TR   |
| (red)  | (blue) |
+--------+--------+
|   BL   |   BR   |
| (green)| (yellow)|
+--------+--------+
```

**Implementation**: Four separate FillRectangle calls per tile

#### 3. Coordinate Calculation
```
destX = tileIndexX × (tileW + 2×padX + 2×spaceX) + spaceX + padX
destY = tileIndexY × (tileH + 2×padY + 2×spaceY) + spaceY + padY
```

**Critical**: The `+ spaceX + padX` offset is essential for correct placement.

## Testing Checklist

- [ ] Command-line parsing (all argument formats)
- [ ] Interactive mode (prompts)
- [ ] All diagonal modes (0-3)
- [ ] Various tile sizes (8×8, 16×16, 32×16, etc.)
- [ ] Edge cases (pad=0, space=0, large padding)
- [ ] Different image formats (PNG, BMP, JPG)
- [ ] Transparency preservation
- [ ] DPI preservation
- [ ] File overwrite confirmation
- [ ] Error handling (missing file, invalid args)
- [ ] Pixel-perfect output vs VB.NET version

## Performance Targets

Based on VB.NET baseline:

| Language | Expected Speedup | Target |
|----------|-----------------|---------|
| Rust     | 5-10x          | 1s for medium tileset |
| C++      | 8-15x          | < 1s for medium tileset |
| Go       | 3-5x           | 1-2s for medium tileset |
| C# (ImageSharp) | 1.5-2x  | 3-4s for medium tileset |
| Python   | 0.3-0.5x       | 15-20s for medium tileset |

**Medium tileset**: 1024×1024 pixels, 32×32 tiles, pad=2, space=1

## Common Pitfalls

### 1. Off-by-One Errors in Mirroring
**Problem**: Easy to get pixel order wrong when reversing

**Solution**: Test with distinctive 2×2 or 4×4 test tiles with different colored pixels

### 2. Incorrect Coordinate Offsets
**Problem**: Forgetting the `+ space + pad` offset in destination calculation

**Solution**: Verify first tile position is exactly at (space + pad, space + pad)

### 3. Integer Division Issues
**Problem**: Different languages handle `/` differently (float vs integer division)

**Solution**: Use explicit integer division: `tileCount = inputWidth // tileWidth`

### 4. Corner Overlap in Mode 3
**Problem**: Incorrect quadrant boundaries cause overlapping colors

**Solution**: Ensure `halfW = tileW / 2` uses integer division, verify no gaps/overlaps

### 5. DPI Not Preserved
**Problem**: Output image has wrong dimensions when opened in some applications

**Solution**: Explicitly copy DPI from input to output bitmap

## Additional Resources

### Original Repository
https://github.com/nobuyukinyuu/tile-extruder

### Relevant Documentation
- Tiled Map Editor: https://doc.mapeditor.org/
- Texture Bleeding Solutions: Various game engine documentation

### Image Processing Concepts
- Bilinear filtering and texture bleeding
- Sprite atlases and texture packing
- Mipmapping and LOD

## Contributing to Documentation

If you find errors or missing information:

1. Document the issue
2. Verify against the VB.NET source code
3. Update the relevant documentation file
4. Test any code examples

## Version History

- **v1.0** (Current): Initial documentation for migration project
  - Comprehensive analysis of VB.NET implementation
  - Language-agnostic specifications
  - Migration recommendations

## Contact

For questions about this documentation or the migration project:
- Check the original TileExtruder repository for issues/discussions
- Refer to the VB.NET source code (`Extruder/Module1.vb`) as authoritative reference

---

## Document Index

Quick links to all documentation:

1. [project-overview.md](./project-overview.md) - What and why
2. [requirements.md](./requirements.md) - What it must do
3. [business-logic.md](./business-logic.md) - How it works
4. [technical-specification.md](./technical-specification.md) - Implementation details
5. [data-flow.md](./data-flow.md) - Data movement and state
6. [examples-and-use-cases.md](./examples-and-use-cases.md) - Usage examples
7. [api-specification.md](./api-specification.md) - Complete reference
8. [migration-guide.md](./migration-guide.md) - How to port

**Start here**: [project-overview.md](./project-overview.md)
**For migration**: [migration-guide.md](./migration-guide.md)
**For reference**: [api-specification.md](./api-specification.md)

---

**Generated**: 2025-11-16
**Source Version**: TileExtruder VB.NET (Visual Studio 2013)
**Purpose**: Migration to high-performance language
