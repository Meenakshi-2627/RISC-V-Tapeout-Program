# RISC-V Reference SoC Tapeout Program

## Table of Contents
- [Introduction](#introduction)
- [Week 0: Environment Setup](#week-0-environment-setup)

## Introduction

The **RISC-V Reference SoC Tapeout Program** is a comprehensive 20-week initiative led by IIT Gandhinagar in collaboration with Synopsys and VSD (VLSI System Design). This program represents India's most ambitious semiconductor mission, designed to empower engineering students with end-to-end chip design expertise—from RTL design to actual silicon fabrication (tapeout).

### Program Overview
- **Duration**: 20 weeks
- **Objective**: Complete hands-on journey from design to silicon tapeout
- **Tools**: Industry-standard Synopsys EDA tools and SCL180 nm PDK
- **Mission**: Contributing to India's semiconductor self-reliance and the "One Tapeout Per Student" vision

### Program Outcomes
Upon completion, participants will have:
- Mastered the complete VLSI design flow from RTL to GDSII
- Gained hands-on experience with industry-grade tools and methodologies
- Developed a fabrication-ready RISC-V System-on-Chip (SoC)
- Acquired skills essential for India's growing semiconductor industry
- Contributed to the national framework for academic silicon tapeouts

This program bridges the critical gap between academic learning and industry requirements, providing authentic VLSI experience that goes beyond simulation to actual chip fabrication.

---

## Week 0: Environment Setup

### Task 1: Ubuntu Installation with Oracle VirtualBox

Successfully installed **Ubuntu 22.04 LTS** on Oracle VirtualBox with the following configuration:
- **Host OS**: Windows 11
- **Virtual Machine**: Oracle VirtualBox 7.0
- **Guest OS**: Ubuntu 22.04.3 LTS
- **Allocated RAM**: 8GB
- **Storage**: 50GB dynamic disk
- **Processor**: 4 cores allocated

The virtual environment provides a stable Linux platform essential for running open-source EDA tools required throughout the tapeout program.

### Task 2: Essential EDA Tools Installation

#### Tool Installation Summary

| Tool | Version | Purpose |
|------|---------|---------|
| Yosys | 0.34+43 | RTL Synthesis | 
| Icarus Verilog | 12.0 | Verilog Simulation | 
| GTKWave | 3.3.104 | Waveform Visualization |

#### 1. Yosys - RTL Synthesis Framework
**Yosys** is an open-source RTL synthesis framework that converts Verilog RTL designs into gate-level netlists. It serves as the foundation for digital synthesis workflows and supports various technology libraries, making it essential for transforming high-level hardware descriptions into implementable circuit representations.


#### 2. Icarus Verilog - Verilog Simulator
**Icarus Verilog** is a comprehensive Verilog simulation and synthesis tool that compiles Verilog source code into executable simulations. It provides accurate behavioral simulation of digital circuits, enabling designers to verify functionality before synthesis and is widely used in academic and professional VLSI design workflows.


#### 3. GTKWave - Waveform Viewer
**GTKWave** is a fully-featured waveform visualization tool that reads VCD, LXT, FST, and other standard simulation dump formats. It provides an intuitive graphical interface for analyzing signal timing, debugging digital circuits, and verifying design behavior through comprehensive waveform analysis capabilities.

---

*This repository will be continuously updated throughout the 20-week program with detailed documentation of each week's tasks, learnings, and achievements.*
---



