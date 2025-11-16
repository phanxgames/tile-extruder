# TileExtruder - Project Overview

## Purpose
TileExtruder is a command-line image processing tool designed to eliminate visual seams in tileset rendering by extruding and padding tile edges.

## Problem Statement
When rendering tilesets in game engines or graphics applications, seams often appear between tiles due to:
- Texture filtering (bilinear, trilinear)
- Floating-point precision issues
- Mipmap generation
- GPU rasterization

## Solution
TileExtruder solves this by:
1. **Extruding** edge pixels outward in a "bidi" (bidirectional) fashion
2. **Spacing** tiles apart to prevent bleeding
3. **Handling corners** with multiple algorithms

## Current Technology Stack
- **Language**: Visual Basic .NET (VB.NET)
- **Framework**: .NET Framework 4.0
- **IDE**: Visual Studio Express 2013
- **Key Dependencies**:
  - System.Drawing (GDI+ for image processing)
  - System.IO (File operations)

## Output Characteristics
- Automatically converts output to 32-bit per pixel (32bpp) format
- Preserves image DPI resolution (horizontal and vertical)
- Maintains original image format (PNG, BMP, etc.)

## Known Limitations
1. Setting pad value greater than tile dimensions results in incorrect extrusion
2. Output is always 32bpp regardless of input format
3. Windows-only due to dependency on System.Drawing/GDI+

## Target Use Case
Primarily used by game developers working with:
- Tiled map editors (e.g., Tiled TMX format)
- Sprite atlases
- Texture packing tools
- 2D game engines requiring pixel-perfect rendering
