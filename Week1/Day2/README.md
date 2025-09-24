# Digital Synthesis and Timing Libraries Guide

## Introduction

This comprehensive guide explores digital synthesis fundamentals, focusing on timing library analysis, Process-Voltage-Temperature (PVT) corners, and various synthesis methodologies for VLSI design optimization.

---

|## Table of Contents|
|----------------------------|
|1. [Introduction to Timing Libraries](#introduction-to-timing-libraries)|
|2. [Understanding PVT Corners](#understanding-pvt-corners)|
|3. [Hierarchical Synthesis](#hierarchical-synthesis)|
|4. [Flat Netlist Synthesis](#flat-netlist-synthesis)|
|5. [Submodule Level Synthesization](#submodule-level-synthesization)|


## Introduction to Timing Libraries

### What is a .lib File in VLSI?

A **.lib file** (Liberty file) serves as a **timing and characterization library** in VLSI design, containing:
- **Behavior, timing, power, and area information** of standard cells
- Gate characteristics for **logic synthesis, place-and-route, and static timing analysis (STA)**
- Essential data for **flip-flops, latches, and combinational logic**

> **Think of .lib as the dictionary of cells** for your design tools

### SKY130 PDK Library Breakdown: `sky130_fd_sc_hd__tt_025C_1v80.lib`

The SKY130 PDK is an **open-source Process Design Kit** based on SkyWater Technology's 130nm CMOS technology.

**Library naming convention:** `sky130_fd_sc_hd__<corner>_<temp>C_<vdd>.lib`

- **tt**: Typical process corner
- **025C**: Operating temperature of 25°C
- **1v80**: Supply voltage of 1.8V

### Opening and Exploring the Library File


> **Note**: To remove syntax highlighting in gvim, use the command `:syn off`

The library file contains crucial parameters:
- **Delay** characteristics and timing arcs
- **Pin power** specifications and leakage
- **Cell power** consumption models
- **Technology** node parameters
- **Area** calculations for placement

---

## Understanding PVT Corners

**PVT = Process, Voltage, Temperature** - Critical variations that determine whether silicon will work across all operating conditions.

### Extended PVT Corner Analysis

| Corner | Process | Voltage | Temperature | Primary Use Case |
|--------|---------|---------|-------------|------------------|
| **SS** | Slow NMOS, Slow PMOS | 1.60V | 125°C | **Worst-case delay** (Setup timing check) |
| **TT** | Typical NMOS, Typical PMOS | 1.80V | 25°C | **Nominal condition** (Standard operation) |
| **FF** | Fast NMOS, Fast PMOS | 1.95V | -40°C | **Best-case delay** (Hold timing check) |
| **SF** | Slow NMOS, Fast PMOS | 1.80V | 25°C | **Stress-test** (Imbalanced devices) |
| **FS** | Fast NMOS, Slow PMOS | 1.80V | 25°C | **Stress-test** (Imbalanced devices) |

### Why PVT Corners Matter

- **Setup timing verification** → Worst-case corner (SS, 1.6V, 125°C)
- **Hold timing verification** → Best-case corner (FF, 1.95V, -40°C)
- **Power analysis** → High voltage + high temperature = worst leakage
- **Design robustness** → Ensures functionality across all conditions

---

## Hierarchical Synthesis

### Design Example: Multiple Modules
```bash
// Sub_module1: AND gate functionality
module sub_module1 (input a, input b, output y);
assign y = a & b;
endmodule

// Sub_module2: OR gate functionality
module sub_module2 (input a, input b, output y);
assign y = a | b;
endmodule

// Top module: Implements y = (a & b) | c
module multiple_modules (input a, input b, input c, output y);
wire net1;
sub_module1 u1(.a(a), .b(b), .y(net1)); // net1 = a & b
sub_module2 u2(.a(net1), .b(c), .y(y)); // y = net1 | c
endmodule
```

### Command Overview

| Command | Function |
|---------|----------|
| `yosys` | Launches the Yosys synthesis tool |
| `read_liberty -lib ../lib/sky130_fd_hd_tt_025C_1v80.lib` | Reads the timing library file |
| `read_verilog multiple_modules.v` | Loads the Verilog design file |
| `synth -top multiple_modules` | Performs synthesis with specified top module |
| `hierarchy -check -top multiple_modules` | **Explicit hierarchy verification (recommended)** |
| `abc -liberty ../lib/sky130_fd_hd_tt_025C_1v80.lib` | Technology mapping using ABC tool |
| `show multiple_modules` | Displays the synthesized netlist graphically |
| `write_verilog multiple_modules_hier.v` | Writes hierarchical netlist to file |

### Viewing Design Files
```bash
gvim multiple_modules.v
```


### Why Hierarchical Synthesis?

**Hierarchical synthesis** follows a **divide and conquer approach**, preserving module instances (u1, u2) and maintaining design structure.

**Advantages:**
- **Faster synthesis** for large designs (incremental compilation)
- **Easier debugging** and readability (submodules preserved)
- **Better reusability** across different designs
- **Modular optimization** with clear boundaries

**Why NAND Implementation is Preferred:**

For basic gates like OR, hierarchical synthesis implements using **NAND and NOT gates** because:
- **NAND implementation** uses stacked NMOS transistors
- **Stacked NMOS** provides better performance than stacked PMOS
- **PMOS devices** are ~2-3× weaker than NMOS (lower mobility)
- **NOR implementation** requires stacked PMOS → higher resistance and slower operation
- **NAND gates scale better** with increasing inputs

---

## Flat Netlist Synthesis

### Command Overview

| Command | Function |
|---------|----------|
| `yosys` | Launches the Yosys synthesis tool |
| `flatten` | **Flattens the hierarchical design structure** |
| `write_verilog multiple_modules_flat.v` | Writes flattened netlist to file |
| `write_verilog -noattr multiple_modules_flat.v` | Writes netlist without attributes |
| `!gvim multiple_modules_flat.v` | Opens flattened netlist for viewing |

> **Note**: In gvim, use `:vsp multiple_modules_hier.v` to view split screen comparison

### Hierarchical vs Flat Synthesis Comparison

| Aspect | Hierarchical | Flat |
|--------|-------------|------|
| **Module Structure** | Preserves submodules | **Single-level design** |
| **Gate Visibility** | Shows as submodule instances | **Clear gate-level structure** |
| **Optimization Scope** | Module-level constraints | **Global cross-module optimization** |
| **Debug Capability** | Module-based analysis | **Direct gate inspection** |
| **Runtime** | Faster for large designs | Slower for complex designs |
| **Memory Usage** | Lower | Higher for large designs |
| **Use Case** | Modular design, reusability | **Maximum optimization** |

### Key Benefits of Flat Synthesis

**Flat synthesis** removes all hierarchy, making the **gate structure clearly visible** and enabling:
- **Aggressive optimization** across module boundaries
- **Comprehensive analysis** of gate-level implementation
- **Simplified downstream processing** for some tools
- **Better area optimization** through logic merging

---

## Submodule Level Synthesization

### Command Overview

| Command | Function |
|---------|----------|
| `yosys` | Launches the Yosys synthesis tool |
| `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib` | Reads the standard cell library |
| `read_verilog multiple_modules.v` | Loads the Verilog design file |
| `synth -top submodule1` | **Synthesizes only the specified submodule** |
| `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib` | Technology mapping for the submodule |
| `show` | Displays the synthesized submodule |

### Strategic Applications of Submodule Synthesis

**Reason 1: Multiple Instance Optimization**
- **Preferred approach** when design contains multiple instances of the same module
- Enables **optimized reuse** of synthesized blocks
- **Reduces synthesis time** through module replication
- **Consistent performance** across identical instances

**Reason 2: Large Design Management**  
- For **massive designs**, submodule synthesis enables **divide and conquer strategy**
- **Improves synthesis runtime** and reduces memory requirements
- **Facilitates parallel synthesis** workflows across design teams
- **Enhanced debug capability** through modular isolation

### Benefits for Complex Digital Systems

This approach significantly enhances:
- **Design scalability** for enterprise-level projects
- **Synthesis performance** through parallel processing
- **Memory efficiency** during compilation
- **Team collaboration** through modular development

---

## Summary

This guide covered essential concepts for digital synthesis:

- **Timing libraries** provide the foundation for synthesis with PVT corner analysis
- **Hierarchical synthesis** offers modularity and faster compilation
- **Flat synthesis** enables maximum optimization at the cost of complexity
- **Submodule synthesis** provides strategic advantages for large-scale designs

Choose the appropriate synthesis approach based on your design requirements, team structure, and optimization goals.
