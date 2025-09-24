## Introduction

This guide covers the fundamentals of digital synthesis, including timing library analysis, hierarchical and flat synthesis approaches, and submodule-level optimization techniques for VLSI design.

---


## Table of Contents
1. [Introduction to Timing Libraries](#introduction-to-timing-libraries)
2. [Hierarchical Synthesis](#hierarchical-synthesis)
3. [Flat Netlist Synthesis](#flat-netlist-synthesis)
4. [Submodule Level Synthesization](#submodule-level-synthesization)

## Introduction to Timing Libraries

### Library File Breakdown: `sky130_fd_hd_tt_025C_1v80.lib`

- **tt**: Typical process corner
- **025C**: Operating temperature of 25°C
- **1v80**: Supply voltage of 1.8V

### Opening the Library File
```bash
gvim ../lib/sky130_fd_hd_tt_025C_1v80.lib
```

> **Note**: To remove syntax highlighting in gvim, use the command `:syn off`

The library file contains crucial parameters including:
- **Delay** characteristics
- **Timing** constraints
- **Pin power** specifications
- **Cell power** consumption
- **Technology** node information
- **Area** calculations

---

## Hierarchical Synthesis

### Command Overview

| Command | Function |
|---------|----------|
| `yosys` | Launches the Yosys synthesis tool |
| `read_liberty -lib ../lib/sky130_fd_hd_tt_025C_1v80.lib` | Reads the timing library file |
| `read_verilog multiple_modules.v` | Loads the Verilog design file |
| `synth -top multiple_module` | Performs synthesis with specified top module |
| `abc -liberty ../lib/sky130_fd_hd_tt_025C_1v80.lib` | Technology mapping using ABC tool |
| `show multiple_modules` | Displays the synthesized netlist graphically |
| `write_verilog multiple_modules_hier.v` | Writes hierarchical netlist to file |
| `gvim multiple_modules_hier.v` | Opens the hierarchical netlist for viewing |
| `write_verilog -noattr multiple_modules_hier.v` | Writes netlist without attributes |
| `!gvim multiple_modules_hier.v` | Shell command to open file in new gvim instance |

### Viewing Design Files
```bash
gvim multiple_modules.v
```

### Why Hierarchical Synthesis?

Hierarchical synthesis **preserves module instances** (u1, u2) and maintains the design structure. For basic gates like OR, it implements using **NAND and NOT gates** because:

- **NAND implementation** uses stacked NMOS transistors
- **Stacked NMOS** provides better performance compared to stacked PMOS
- **NOR implementation** would require stacked PMOS, which is less efficient

This approach offers superior **speed and power characteristics** in CMOS technology.

---

## Flat Netlist Synthesis

### Command Overview

| Command | Function |
|---------|----------|
| `yosys` | Launches the Yosys synthesis tool |
| `flatten` | Flattens the hierarchical design |
| `write_verilog multiple_modules_flat.v` | Writes flattened netlist to file |
| `!gvim multiple_modules_flat.v` | Opens the flattened netlist for viewing |
| `write_verilog -noattr multiple_modules_flat.v` | Writes netlist without attributes |
| `!gvim multiple_modules_flat.v` | Shell command to open file in new instance |

> **Note**: In gvim, to view split screen of two files, use: `:vsp multiple_modules_hier.v`

### Hierarchical vs Flat Synthesis

| Aspect | Hierarchical | Flat |
|--------|-------------|------|
| **Structure** | Preserves submodules | Single-level design |
| **Visibility** | Shows as submodule instances | **Clear gate-level structure** |
| **Debugging** | Module-based analysis | Direct gate inspection |
| **Optimization** | Module-level constraints | **Global optimization** |

**Flat synthesis** removes all hierarchy, making the **gate structure clearly visible** and enabling comprehensive analysis of the implementation.

---

## Submodule Level Synthesization

### Command Overview

| Command | Function |
|---------|----------|
| `yosys` | Launches the Yosys synthesis tool |
| `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib` | Reads the standard cell library |
| `read_verilog multiple_modules.v` | Loads the Verilog design file |
| `synth -top submodule1` | Synthesizes only the specified submodule |
| `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib` | Technology mapping for the submodule |
| `show` | Displays the synthesized submodule |

### Why Submodule Level Synthesis?

**Reason 1: Multiple Instances**
- Preferred when design contains **multiple instances** of the same module
- Enables **optimized reuse** of synthesized blocks

**Reason 2: Massive Designs**
- For **large-scale designs**, submodule synthesis enables **divide and conquer** strategy
- Improves **synthesis runtime** and **memory efficiency**
- Facilitates **parallel synthesis** workflows

This approach significantly enhances **design scalability** and **synthesis performance** for complex digital systems.



