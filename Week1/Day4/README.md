# Day 4 - Gate-Level Simulation (GLS), Blocking vs Non-blocking and Synthesis-Simulation Mismatch

This document covers Gate-Level Simulation (GLS) to verify post-synthesis behavior, and explores common Synthesis-Simulation mismatches caused by sensitivity lists and blocking vs non-blocking assignments in Verilog.

## Table of Contents

1. [Gate-Level Simulation (GLS)](#gate-level-simulation-gls)
   - [What is GLS?](#what-is-gls)
   - [Why Perform GLS?](#why-perform-gls)
   - [Types of GLS](#types-of-gls)
   - [GLS Workflow](#gls-workflow)
   - [Challenges](#challenges)

2. [Synthesis-Simulation Mismatch](#synthesis-simulation-mismatch)
   - [Missing Sensitivity List](#missing-sensitivity-list)
   - [Blocking vs Non-blocking Assignments](#blocking-vs-non-blocking-assignments)
   - [Caveats with Blocking Statements](#caveats-with-blocking-statements)

3. [Labs on Gate-Level Simulation](#labs-on-gate-level-simulation)
   - [Lab 1: Ternary Operator MUX](#lab-1-ternary-operator-mux)
   - [Lab 2: Bad MUX Example](#lab-2-bad-mux-example)
   - [Lab 3: Blocking Caveat](#lab-3-blocking-caveat)

4. [Summary](#summary)

---

## Gate-Level Simulation (GLS)

### What is GLS?

**Gate-Level Simulation (GLS)** is a verification technique that simulates the synthesized gate-level netlist instead of the original RTL code. It validates the functionality and timing behavior of a digital circuit after synthesis, using actual standard cells and their associated delays.

- **RTL Simulation**: High-level behavioral simulation
- **GLS**: Low-level gate-level simulation with actual hardware models

### Why Perform GLS?

GLS is essential for comprehensive verification because it:

- **Validates Synthesis**: Ensures synthesis tools correctly translated RTL into gates
- **Catches Mismatches**: Identifies differences between RTL and gate-level behavior
- **Timing Verification**: Simulates with realistic delays using SDF (Standard Delay Format) files
- **Reset Verification**: Confirms proper reset behavior and initialization
- **DFT Validation**: Verifies scan chains and other Design-for-Test structures
- **X-Propagation**: Checks handling of unknown states (X) in the design

### Types of GLS

1. **Functional GLS**: 
   - Zero-delay simulation
   - Focuses on logic functionality
   - Checks for synthesis-simulation mismatches

2. **Unit-Delay GLS**:
   - All gates have unit delay
   - Exposes race conditions
   - Better timing awareness than zero-delay

3. **Timing GLS**:
   - Uses SDF back-annotation
   - Real timing validation with actual delays
   - Most accurate representation of hardware behavior

### GLS Workflow

RTL Design + Testbench → Synthesis → Gate-level Netlist
↓
Gate-level Netlist + Verilog Models + Testbench → GLS → Verification

**Required Files for GLS**:
- Synthesized gate-level netlist (.v)
- Standard cell library models (primitives.v, sky130_fd_sc_hd.v)
- Original testbench
- SDF file (for timing simulation)

### Challenges

- **Complexity**: More detailed gate-level representation increases simulation time
- **Memory Requirements**: Higher memory usage compared to RTL simulation
- **Debugging Difficulty**: Harder to trace issues in gate-level netlist
- **Initialization Problems**: X-propagation and unknown state handling
- **Setup Requirements**: Need proper library models and timing files

---

## Synthesis-Simulation Mismatch

A **synthesis-simulation mismatch** occurs when RTL simulation results differ from gate-level simulation or actual hardware behavior. Common causes include:

### Missing Sensitivity List

**Problem**: Incomplete sensitivity lists in combinational always blocks cause simulation-only errors.
```
// Problematic code
always @(a) begin
y = a & b; // 'b' missing from sensitivity list
end
```

**Issue**: Simulator won't re-evaluate when `b` changes, but synthesis assumes full sensitivity.

**Solution**: Use `always @(*)` for combinational logic:
```
// Corrected code
always @(*) begin
y = a & b; // All inputs automatically included
end
```
### Blocking vs Non-blocking Assignments

#### Blocking Assignment (`=`)
- **Execution**: Sequential, immediate
- **Use Case**: Combinational logic
- **Behavior**: Statements execute in order, one after another
```
always @(*) begin
y = a & b; // Executes immediately
z = y | c; // Uses updated value of y
end
```
#### Non-blocking Assignment (`<=`)
- **Execution**: Concurrent, scheduled at end of time step
- **Use Case**: Sequential logic (clocked)
- **Behavior**: All RHS evaluated first, then LHS updated simultaneously
```
always @(posedge clk) begin
q <= d; // Old values used for evaluation
r <= q; // r gets old value of q (shift register behavior)
end
```
#### Comparison Table

| **Aspect** | **Blocking (`=`)** | **Non-blocking (`<=`)** |
|------------|-------------------|-------------------------|
| **Execution** | Sequential, immediate | Concurrent, end of timestep |
| **Best for** | Combinational logic | Sequential logic |
| **Timing** | Order-dependent | Order-independent |
| **Hardware Model** | Combinational gates | Flip-flops/latches |

### Caveats with Blocking Statements

**Problem**: Using blocking assignments in sequential logic can cause race conditions and mismatches.
```
// Problematic: Blocking in sequential logic
always @(posedge clk) begin
q = d;
r = q; // r gets new value of q immediately
end
```

**Issue**: Creates unintended combinational behavior instead of proper sequential logic.

**Solution**: Use non-blocking for sequential logic:
```
// Correct: Non-blocking for sequential logic
always @(posedge clk) begin
q <= d;
r <= q; // r gets old value of q (proper shift register)
end
```
---

## Labs on Gate-Level Simulation

### Lab 1: Ternary Operator MUX

**Objective**: Demonstrate GLS flow with a simple multiplexer design.

#### Verilog Code
```
module ternary_operator_mux (input i0, input i1, input sel, output y);
assign y = sel ? i1 : i0;
endmodule
```
#### RTL Simulation
```
iverilog -o ternary_mux.out ternary_operator_mux.v tb_ternary_operator_mux.v
./ternary_mux.out
gtkwave tb_ternary_operator_mux.vcd
```
#### Synthesis
```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog ternary_operator_mux_net.v
show
```

#### Gate-Level Simulation
```
iverilog -o ternary_mux_gls.out
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
ternary_operator_mux_net.v
tb_ternary_operator_mux.v
./ternary_mux_gls.out
gtkwave tb_ternary_operator_mux.vcd
```

**Result**: RTL and GLS waveforms should match exactly, confirming no synthesis-simulation mismatch.

---

### Lab 2: Bad MUX Example

**Objective**: Demonstrate synthesis-simulation mismatch due to incomplete sensitivity list.

#### Verilog Code
```
module bad_mux (input i0, input i1, input sel, output reg y);
always @(sel) // Missing i0, i1 in sensitivity list
begin
if(sel)
y <= i1;
else
y <= i0;
end
endmodule
```

#### Analysis
- **RTL Behavior**: Only responds to `sel` changes, ignoring `i0`/`i1` changes
- **Synthesis Result**: Creates proper mux responding to all inputs
- **Outcome**: Mismatch between RTL and GLS simulations

#### RTL Simulation
```
iverilog -o bad_mux.out bad_mux.v tb_bad_mux.v
./bad_mux.out
gtkwave tb_bad_mux.vcd
```
#### Synthesis
```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_mux.v
synth -top bad_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog bad_mux_net.v
show
```
#### GLS Simulation
```
iverilog -o bad_mux_gls.out
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
bad_mux_net.v
tb_bad_mux.v
./bad_mux_gls.out
gtkwave tb_bad_mux.vcd
```

**Key Observation**: RTL simulation shows output not changing when inputs change (only when sel changes), while GLS shows proper mux behavior.

**Key Learning**: Always use `always @(*)` for combinational logic to avoid sensitivity list issues.

---

### Lab 3: Blocking Caveat

**Objective**: Demonstrate issues with blocking statements in combinational logic.

#### Verilog Code
```
module blocking_caveat (input a, input b, input c, output reg d);
reg x;
always @(*) begin
d = x & c; // Uses old value of x
x = a | b; // Updates x after d is calculated
end
endmodule
```

#### Analysis
- **RTL Behavior**: `d` uses previous value of `x` due to blocking assignment order
- **Synthesis Result**: `d = (a | b) & c` - correct combinational logic
- **Outcome**: Timing mismatch between RTL and GLS

#### RTL Simulation
```
iverilog -o blocking_caveat.out blocking_caveat.v tb_blocking_caveat.v
./blocking_caveat.out
gtkwave tb_blocking_caveat.vcd
```

#### Synthesis
```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog blocking_caveat_net.v
show
```
#### GLS Simulation
```
iverilog -o blocking_caveat_gls.out
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
blocking_caveat_net.v
tb_blocking_caveat.v
./blocking_caveat_gls.out
gtkwave tb_blocking_caveat.vcd
```

#### Solution Options
```
1. **Reorder statements**:
always @(*) begin
x = a | b; // Calculate x first
d = x & c; // Then use updated x
end
```
2. **Use continuous assignment**:
```
assign x = a | b;
assign d = x & c;
```

**Key Learning**: Be careful with blocking statement order in combinational logic.

---

## Summary

### Gate-Level Simulation (GLS)
- **Purpose**: Validates post-synthesis netlist functionality, timing, and reset behavior
- **Types**: Functional (zero-delay), Unit-delay, and Timing simulation (SDF)
- **Benefits**: Catches synthesis-simulation mismatches, timing violations, and initialization issues
- **Challenges**: Increased complexity, simulation time, and debugging difficulty

### Synthesis-Simulation Mismatch Prevention
- **Sensitivity Lists**: Use `always @(*)` for combinational logic to include all relevant signals
- **Assignment Types**: 
  - Blocking (`=`) for combinational logic - sequential execution
  - Non-blocking (`<=`) for sequential logic - concurrent execution
- **Best Practices**: Write clear, unambiguous, synthesizable RTL code

### Key Coding Guidelines
1. **Combinational Logic**: Use `always @(*)` with blocking assignments (`=`)
2. **Sequential Logic**: Use `always @(posedge clk)` with non-blocking assignments (`<=`)
3. **Avoid**: Mixing blocking and non-blocking in the same always block
4. **Testing**: Always verify both RTL and gate-level behavior

### Lab Summary
- **Lab 1 (Ternary MUX)**: Clean design with no mismatches - good baseline
- **Lab 2 (Bad MUX)**: Demonstrates sensitivity list issues and their impact
- **Lab 3 (Blocking Caveat)**: Shows blocking assignment order dependency problems

> **Note**: Despite being time-consuming and challenging, GLS significantly improves design quality and confidence, uncovering issues that RTL simulation might miss. It's essential for robust VLSI design verification.

---

## Best Practices for GLS

1. **Always simulate both RTL and gate-level netlists**
2. **Use proper Verilog coding guidelines**
3. **Maintain consistent file organization**
4. **Document any known limitations or assumptions**
5. **Review synthesis and simulation tool warnings carefully**

*This documentation provides a comprehensive guide to Gate-Level Simulation and common pitfalls in Verilog RTL design. Follow these practices for robust, reliable digital designs.*
