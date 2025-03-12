---
layout: post
---

# RBXE - A Pixel Game Engine with Voxel Power

## Introduction

Game engines are the foundation of game experiences, providing developers with the tools to create fun and engaging worlds. While modern engines like Unity and Unreal offer powerful capabilities, there's something deeply satisfying about building a game engine from scratch. This is why I created **RBXE**, a pixel game engine designed for simplicity, flexibility, and retro-style game development.

In this article, I'll walk you through why I built RBXE, how it works, and how you can use it to create your own games.

## Why I Built RBXE

As a retro game enthusiast and developer, I wanted an engine that focused on **pixel graphics** and **classic game** mechanics without the overhead of complex modern frameworks. Many existing engines are either too large or lack the low-level control I needed. With RBXE, I aimed to create a C based, lightweight yet powerful "engine", tailored for pixel-based games, allowing me to focus on creativity rather than boilerplate code and platform issues.

## Features of RBXE

- **Lightweight**: Minimal dependencies and efficient rendering.
- **Cross-platform**: Runs on multiple platforms with minimal setup.
- **Simple API**: Easy-to-use functions for drawing, input handling, and game logic.
- **Customizable**: Easy extendable for adding custom features.
- **Retro-style rendering**: Perfect for 8-bit and 16-bit inspired games.
- **Voxel Rendering Support**: Basic voxel-space based rendering.

## How RBXE Works

At its core, RBXE handles the essential components of a game engine:

- **Rendering**: Efficiently draws pixel-based graphics using a custom framebuffer.
- **Game Loop**: Provides a structured update and render cycle for smooth performance.
- **Input Handling**: Captures keyboard and mouse inputs for player interaction.
- **Asset Management**: Supports loading and managing sprites, tiles, and animations.
- **Pixel Rendering**: Implements a basic pixel-based rendering technique.

### The Voxel Space Technique Used in Games Like Comanche

Voxel space rendering, popularized by games like Comanche: Maximum Overkill in the early 1990s, differs from traditional polygon-based rendering by representing the game world as a heightmap-based voxel terrain rather than using 3D polygons.

![Comanche Voxel Space](/images/comanche-1.gif)

### Key Concepts of the Voxel Space Technique:

- Heightmap-based terrain: A 2D array where each value represents terrain elevation.
- Raycasting for rendering: Instead of rendering cubes, a ray is cast across the heightmap, determining visibility and perspective.
- Perspective correction: Adjustments are made to simulate a pseudo-3D effect based on depth and height relationships.
- Efficient memory use: Unlike polygon-based models, the terrain does not require complex vertex data, making it ideal for large landscapes.

Voxel-based rendering allowed Comanche to feature smooth, rolling terrains, unlike the blocky, polygonal landscapes seen in other early 3D games. The approach used ray height projection to determine which terrain pixels should be rendered at what height, ensuring a fluid and immersive visual experience.

### How RBXE Implements a Voxel Space Renderer

RBXE's voxel renderer follows a similar principle by using raycasting over a heightmap to create a voxel terrain with real-time rendering. The engine projects terrain height based on camera position, distance, and depth-based shading, producing a pseudo-3D landscape.

This approach allows for:

- **Fast rendering of large terrains** without requiring complex geometry.
- **Smooth terrain scrolling** with real-time height-based adjustments.
- **Efficient memory usage** while retaining high visual fidelity.

## Exploring the RBXE Voxel Renderer

The voxel space rendering in RBXE is based on heightmaps and colormaps loaded from GIF images. The engine extracts height and color information and renders a 3D perspective scene by iterating through the heightmap and projecting pixels onto the screen.

### Key Components of the Voxel Renderer:

- **Camera System**: Tracks movement, height, and rotation.
- **Ray Projection**: Calculates terrain heights and perspective projection.
- **Efficient Rendering**: Draws only visible terrain, reducing computational load.

![Voxel Space Rendering](/images/voxel.gif)

#### Code Breakdown

Here’s a simplified version of how the voxel space rendering works:

```C
void render_voxel(void) {
    float sinangle = sin(camera.angle);
    float cosangle = cos(camera.angle);

    float plx = cosangle * camera.zfar + sinangle * camera.zfar;
    float ply = sinangle * camera.zfar - cosangle * camera.zfar;
    float prx = cosangle * camera.zfar - sinangle * camera.zfar;
    float pry = sinangle * camera.zfar + cosangle * camera.zfar;

    for (int i = 0; i < SCREEN_WIDTH; i++) {
        float deltax = (plx + (prx - plx) / SCREEN_WIDTH * i) / camera.zfar;
        float deltay = (ply + (pry - ply) / SCREEN_WIDTH * i) / camera.zfar;

        float rx = camera.x;
        float ry = camera.y;
        float tallestheight = SCREEN_HEIGHT;

        for (int z = 1; z < camera.zfar; z++) {
            rx += deltax;
            ry += deltay;

            int mapoffset = ((MAP_N * ((int)(ry) & (MAP_N - 1))) + ((int)(rx) & (MAP_N - 1)));
            int projheight = (int)((camera.height - heightmap[mapoffset]) / z * SCALE_FACTOR + camera.horizon);

            if (projheight < tallestheight) {
                for (int y = projheight; y < tallestheight; y++) {
                    if (y >= 0) {
                        int pal = colormap[mapoffset];
                        pixel_info color = {
                            palette[pal * 3 + 0] * LIGHT,
                            palette[pal * 3 + 1] * LIGHT,
                            palette[pal * 3 + 2] * LIGHT,
                            255
                        };
                        pixbuf[(SCREEN_WIDTH * (SCREEN_HEIGHT - y)) + i] = color;
                    }
                }
                tallestheight = projheight;
            }
        }
    }
}
```

## Conclusion

The RBXE Voxel Space Renderer is a step toward expanding the engine's capabilities while maintaining a retro-inspired aesthetic. By combining heightmaps and raycasting, it achieves a fast, efficient rendering method ideal for classic-style 3D landscapes.

If you’re interested in exploring voxel rendering with RBXE, check out the GitHub repository and experiment with the code!

[RBXE on GitHub](https://github.com/rogerboesch/rbxe)