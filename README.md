# VGA Bouncing Ball in Verilog

VGA technology works by transmitting a video signal that includes separate red, green, and blue color signals along with horizontal and vertical sync signals from devices like computers and DVD players to displays such as monitors and projectors.

---

## Table of Contents
1. Introduction
2. VGA Standard
3. System Overview
4. Module Explanation
5. Simulation (Manual)

---

## 1. Introduction

This project implements a minimal VGA graphics system in Verilog.  
It generates a 640×480 @60Hz VGA signal and displays a bouncing ball animation.

The image is not stored in memory.  
Instead, it is generated pixel by pixel in real time using simple logic.

The design consists of three modules connected in sequence:

vga_controller → graphics → display → VGA output

Each module performs a specific role in generating the video signal.

---

## 2. VGA Standard

A VGA monitor does not display the whole image at once.  
It draws the screen pixel by pixel using a process called raster scanning.

The scanning sequence is:

1. Start at the top-left pixel.
2. Move left to right across the row.
3. When the row ends, move down one line.
4. Repeat until the bottom of the screen.

### Scan pattern

(0,0) → (1,0) → (2,0) → ... → (639,0)  
↓  
(0,1) → (1,1) → (2,1) → ... → (639,1)  
↓  
(0,2) → ...

Each pixel is generated on every clock cycle.

---

### VGA Timing (640×480 @60Hz)

![VGA Timing](docs/images/vga_timing.png)


### Total pixels per frame

800 × 525 = 420,000 pixels

At 60 frames per second:

420,000 × 60 ≈ 25.175 MHz pixel clock

So the design operates at approximately a 25 MHz pixel clock.

---

## 3. System Overview

The system generates the video signal using three modules:

vga_controller → graphics → display → VGA output

For every clock cycle:

1. The VGA controller produces the current pixel position.
2. The graphics module determines the pixel color.
3. The display module forwards the signals to the output.

This process repeats continuously to form the image.

---
## 4. Module Explanation

### vga_controller

This module generates all VGA timing signals.

It produces:

- Horizontal pixel coordinate (`coord_x`)
- Vertical pixel coordinate (`coord_y`)
- Horizontal sync (`hsync`)
- Vertical sync (`vsync`)
- Active display signal (`active_area`)

It uses two counters:

- Horizontal counter: counts from 0 to 799  
  - Visible region: 0 to 639
- Vertical counter: counts from 0 to 524  
  - Visible region: 0 to 479

The `active_area` signal indicates whether the current pixel is inside the visible screen:

- `active_area = 1` → visible pixel  
- `active_area = 0` → blanking region  

Graphics are rendered only when `active_area` is high.

---

### graphics module

This module determines the color of each pixel.

Inputs:

- `coord_x`
- `coord_y`
- `active_area`

Output:

- `rgb`

Different versions of the `graphics` module can be used to generate
different patterns or animations on the VGA screen.

---

### display module

This is the top-level module.

It connects:

vga_controller → graphics → VGA outputs

It is responsible only for wiring the modules together and forwarding
the RGB and sync signals to the output.

---

## 5. Simulation (Manual)

### Compile and Run

Install everything required to run this simulation on Ubuntu Linux: 

```bash
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install verilator
sudo apt-get install libglu1-mesa-dev freeglut3-dev mesa-common-dev
```
Open terminal in the rtl folder and compile design :
```
verilator -Wall --cc --exe simulator.cpp display.v -LDFLAGS -lglut -LDFLAGS -lGLU -LDFLAGS -lGL
```

Build the executable:
```
make -j -C obj_dir -f Vdisplay.mk Vdisplay
```
Run the simulation:
```
obj_dir/Vdisplay
```

---

## Simulation Result
<div align="center">

<table>
<tr>
<td align="center">

**VGA Output**

<img src="results/vga_output.png" width="300">

</td>

<td width="40"></td>

<td align="center">

**Simulation Animation**

<img src="results/vga_demo.gif" width="300">

</td>
</tr>
</table>

</div>

---

## Other Graphics Examples

Additional pattern examples are provided as separate files.

To try a different pattern:

1. Open the desired pattern file.
2. Copy its contents.
3. Replace the contents of `graphics.v` with that code.
4. Recompile and run the simulation.

This allows you to quickly test different graphics patterns on the VGA screen.
>>>>>>> 919ef17 (added new patterns and result of simulation)

---

## Other Graphics Examples

Additional pattern examples are provided as separate files.

To try a different pattern:

1. Open the desired pattern file.
2. Copy its contents.
3. Replace the contents of `graphics.v` with that code.
4. Recompile and run the simulation.

This allows you to quickly test different graphics patterns on the VGA screen.

