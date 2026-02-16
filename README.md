# VGA Bouncing Ball in Verilog

This project demonstrates a **minimal VGA graphics system** written in Verilog.  
It generates a **640×480 @60Hz VGA signal** and displays a **bouncing ball animation**.

The goal of this project is to explain:

- How VGA works
- How pixels are generated
- How graphics are created in hardware
- How animation works in RTL

This is a **fully procedural graphics design**.  
No framebuffers or image memory are used.

---

## Project Structure

rtl/ → design files (display, controller, graphics)
sim/ → testbench and run script
README.md


---

## How VGA Works

A VGA monitor does not display the entire image at once.  
It draws the screen **pixel by pixel**.

It scans the screen like this:

1. Start at the top-left corner
2. Move left to right
3. Draw one line
4. Move down
5. Repeat until the bottom

This is called **raster scanning**.

---

### Screen Scan Pattern

(0,0) → (1,0) → (2,0) → ... → (639,0)
↓
(0,1) → (1,1) → (2,1) → ... → (639,1)
↓
(0,2) → ...


One pixel is drawn **every clock cycle**.

---

## VGA Timing (640×480 @60Hz)

Below is the industry-standard timing for VGA:

![VGA Timing](docs/images/vga_timing.png)

---

### General Timing

| Parameter | Value |
|----------|------|
| Resolution | 640×480 |
| Refresh rate | 60 Hz |
| Vertical frequency | 31.46875 kHz |
| Pixel clock | 25.175 MHz |

---

### Horizontal Timing (one line)

| Section | Pixels |
|--------|--------|
| Visible area | 640 |
| Front porch | 16 |
| Sync pulse | 96 |
| Back porch | 48 |
| **Total** | **800** |

Horizontal counter range:

0 → 799


---

### Vertical Timing (one frame)

| Section | Lines |
|--------|-------|
| Visible area | 480 |
| Front porch | 10 |
| Sync pulse | 2 |
| Back porch | 33 |
| **Total** | **525** |

Vertical counter range:

0 → 524


---

### Total Pixels per Frame

800 × 525 = 420,000 pixels


At 60 frames per second:

420,000 × 60 ≈ 25 MHz pixel clock


So the system clock is:

≈ 25 MHz


---

## System Architecture

The design uses three modules:

vga_controller → graphics → display → screen


---

## Data Flow (every clock cycle)

1. Clock ticks
2. VGA controller updates pixel position
3. Graphics module decides pixel color
4. Display module sends color to output

---

## Module Overview

### vga_controller
This module generates:

- Pixel coordinates
- Sync signals
- Active display region

It uses two counters.

#### Horizontal counter
Counts pixels across one line:

0 → 799


Visible area:

0 → 639


#### Vertical counter
Counts lines down the screen:

0 → 524


Visible area:

0 → 479


#### active_area signal
active_area = 1 → visible pixel
active_area = 0 → blanking region


Graphics should only draw when:

active_area == 1


---

### graphics module
This module decides:

What color should this pixel be?


Inputs:

coord_x
coord_y
active_area


Output:

rgb


---

## Ball Drawing Logic

The ball is drawn using a circle equation.

For each pixel:

dx = pixel_x − ball_center_x
dy = pixel_y − ball_center_y

If:
dx² + dy² < radius²
→ pixel is inside ball


If inside ball:

rgb = blue


Else:

rgb = black


---

## Ball Animation Logic

The ball position is stored in registers:

ball_x
ball_y


A counter slows down movement.

Every few clock cycles:

ball_x += speed
ball_y += speed


When the ball hits a screen edge:

direction reverses


So the ball appears to **bounce**.

---

### display module
This is the top-level module.

It connects:

vga_controller → graphics → outputs


It does not:

- Generate timing
- Draw shapes
- Store images

It only connects modules.

---

## Full Timing Sequence

For each clock cycle:

Clock tick
↓
coord_x increases
↓
When coord_x reaches end:
coord_x resets
coord_y increases
↓
When coord_y reaches end:
new frame starts


This repeats **60 times per second**.

---

## Animation Speed Example

If:

Clock = 25 MHz
Counter threshold = 200,000


Then:

25,000,000 / 200,000 = 125 updates per second


So the ball updates at:

≈125 times per second


---

## Simulation Instructions

### Install tools
```bash
sudo apt install iverilog gtkwave
Run simulation
From project root:

cd sim
./run.sh
This will:

Compile the design

Run the simulation

Open waveforms

Manual run (without script)
iverilog -o sim.out \
rtl/display.v \
rtl/vga_controller.v \
rtl/graphics.v \
sim/tb_display.v

