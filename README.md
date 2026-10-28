# Tesseract

A small [raylib](https://www.raylib.com/) program that draws a rotating **tesseract** (a 4D hypercube) projected into 3D, which you can fly around with a free camera.

Everything is in one C file, `main.c`, and the only dependency is raylib.

> Related project: **[Robocon-2027](https://github.com/TariatiinEserguu/Robocon-2027)**, the ESP32 firmware for our Robocon R1 robot. The two repos have the same author and are used for practice: this one for 3D/4D math and graphics, that one for embedded control.

---

## What it does

- Defines the 16 vertices of a tesseract in 4D (`x, y, z, w` = ±1).
- Each frame it rotates them in **two planes at once**: **XY** and **ZW**. This is a "double rotation", which only exists in 4D.
- It projects every 4D point down to 3D with a perspective divide on `w`:

  ```
  k   = 1 / (2 - w)
  p3D = (x·k, y·k, z·k) · 2 + (0, 5, 0)
  ```

  Points with larger `w` are "closer" in the 4th dimension, so they look bigger.
- It draws the result with raylib's 3D helpers:
  - 🔴 **red**: the cube at `w = +1` (vertices and 12 edges)
  - 🔵 **blue**: the cube at `w = -1` (vertices and 12 edges)
  - 🟢 **green**: the 8 edges that join matching corners of the two cubes

  That is all **16 vertices and 32 edges** of a tesseract. A reference grid is drawn on the floor.

## Controls

| Key / input           | Action                                     |
|-----------------------|--------------------------------------------|
| Mouse + `W A S D`     | Free camera (raylib `CAMERA_FREE`)         |
| Mouse wheel           | Zoom                                       |
| `R` / `F`             | Move the camera up / down                  |
| `Z`                   | Point the camera back at the origin        |
| `Space`               | Pause / resume the 4D rotation             |
| `Esc`                 | Quit                                       |

## Building

You need **raylib 4.5 or newer**, because `UpdateCamera(&camera, CAMERA_FREE)` takes a camera mode. Older 4.x versions use `SetCameraMode` instead.

### Linux

```bash
gcc main.c -o tesseract -lraylib -lGL -lm -lpthread -ldl -lrt -lX11
```

```bash
./tesseract
```

### macOS (Homebrew)

```bash
brew install raylib
```

```bash
cc main.c -o tesseract $(pkg-config --cflags --libs raylib)
```

### Windows (w64devkit / MinGW, as shipped with the raylib installer)

```bash
gcc main.c -o tesseract.exe -lraylib -lopengl32 -lgdi32 -lwinmm
```

## How the code is laid out

| Part of `main.c`      | What it does                                                         |
|-----------------------|----------------------------------------------------------------------|
| `points[16]`          | The tesseract vertices in 4D (`Vector4`)                             |
| `indices[12]`         | Edge list of one cube. It is reused for both cubes (offset by 8)     |
| `mat4_mul_vec4`       | Multiplies a raylib `Matrix` by a `Vector4`                          |
| `mat43_mul_vec4`      | 3×4 matrix × `Vector4`, used for the 4D → 3D projection              |
| `update()`            | Advances the angle, rotates in XY and ZW, and projects to `points_3` |
| `draw_tesseract()`    | Draws the spheres (vertices) and lines (edges)                       |
| `main()`              | Window, camera, input, render loop (800×600, 60 FPS)                 |

## Ideas to try

- Rotate in other planes (XW, YW) or use a different speed for each plane.
- Change the projection distance (`2 - w`) and watch the inner cube grow or shrink.
- Swap the 4D perspective projection for an orthographic one (simply drop `w`).
- Draw the 24 square faces with transparency instead of only edges.

## License

No license has been chosen yet. Until one is added, all rights are reserved by the author.
