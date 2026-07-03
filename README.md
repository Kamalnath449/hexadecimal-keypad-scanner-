# 4x4 Hexadecimal Keypad Scanner & Encoder

An RTL-level implementation in SystemVerilog of a complete structural scanning and decoding circuit for a standard $4 \times 4$ Matrix Keypad (modeled after the Grayhill 072 series). 

This project implements sequential hardware column-scanning logic to pinpoint mechanical keypresses across a 16-key matrix, stabilizes the inputs to prevent metastability/bounce, and outputs a clean 4-bit binary code along with a data-valid handshake strobe.

---

## 📐 Hardware Specifications & Features

* **Matrix Interfacing**: Dynamically drives 4 Column output lines (`Col`) and samples 4 Row input lines (`Row`) to save physical I/O pins, controlling 16 distinct buttons using only 8 physical pins.
* **Asynchronous Stabilization**: Includes active input synchronization to prevent hardware metastability and button-bounce glitches from propagating into downstream control registers.
* **Synchronous Binary Output**: Decodes active physical row/column intersections into a normalized hexadecimal code value (`4'h0` to `4'hF`).
* **Handshake Strobe**: Asserts a `Valid` signal line when a keypress event is stabilized, ensuring reliable data capture for downstream logic processing.

---

## 🏗️ Structural File Hierarchy

The architecture avoids a single massive file block by adhering to clean, industry-standard modular design boundaries:

* **design.sv**: The top-level compilation wrapper that bundles the physical sub-blocks together using compiler include directives.
* **synchronize.sv**: A multi-stage flip-flop shift register block designed to isolate external asynchronous mechanical contacts into a safe, clock-aligned domain.
* **hex_keypad_grayhill072.sv**: The fundamental finite state machine (FSM) and scanning controller that sequences column logic and monitors row line feedback.
* **row_signal.sv**: A specialized decoder sub-module that translates raw physical multi-bit switch paths into cleanly mapped signal vectors.
* **testbench.sv**: A comprehensive simulation environment that models matrix contact behaviors, driving virtual button test vectors through a standard one-hot encoded matrix model (`Key[15:0]`).

---

## ⚙️ Core Operational Logic

The scanner works by continuously rotating an active-low or active-high pattern across the column vector (`Col[3:0]`). 

When a button at intersection $(r, c)$ is mechanically pressed, the logic state of column $c$ bridges directly onto row $r$. The encoder then uses the active coordinates to index the matching hex representation:

| Button Identity | Column Line Index ($c$) | Row Line Index ($r$) | Encoded Output (`Code[3:0]`) |
|:---:|:---:|:---:|:---:|
| **0** | Column 0 | Row 0 | `4'h0` |
| **5** | Column 1 | Row 1 | `4'h5` |
| **A** | Column 2 | Row 2 | `4'hA` |
| **F** | Column 3 | Row 3 | `4'hF` |

---

## 🛠️ Verification & Simulation Usage

The testbench is configured to execute via modern EDA simulation suites using standard SystemVerilog toolchains.

### Simulation Guidelines:
1. Ensure the simulation parameters are defined with an appropriate simulation timescale profile (e.g., `` `timescale 1ns/1ps ``).
2. Explicitly size your testbench variables to prevent implicit 1-bit net inference mismatches:
   ```systemverilog
   wire [15:0] Key; // 16-bit virtual stimulus
   wire [3:0]  Col;
   wire [3:0]  Row;
