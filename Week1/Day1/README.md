# Introduction to Verilog RTL Design and Synthesis

This day covers the topic of iverilog simulation and yosys 

>Note :
>Before proceeding with Verilog RTL design and synthesis, ensure the following tools are installed:
>- You have a Linux environment (Ubuntu 22.04 recommended).
>- Icarus Verilog simulator is installed and accessible.
>- GTKWave waveform viewer is configured.
>- Basic understanding of digital logic concepts.

## Table of Contents :
[1. Introduction to iverilog simulator ](#introduction-to-open-source-simulator-iverilog)
 - [What is Simulator , Design , Testbench?](#simulator)
 - [Simulation flow of iverilog](#iverilog-based-simulation-flow)
 - [D Latch execution- .v file](#latch-execution)
 - [Tool Setup](#environmental-setup)

# Introduction to Open-Source Simulator: iverilog

### Simulator

A **simulator** is a software tool that mimics the behavior of digital circuits by executing Verilog code and tracking signal changes over time.

Key characteristics of simulators include:

- Simulator monitors input signals and triggers output evaluation only when inputs change.
- No change in input signals results in no change in output signals.
- The simulator applies stimulus (set of test signals) to the design to test its behavior.

### Design

A **design** is the actual Verilog module that implements the intended digital functionality and contains primary inputs and outputs.

### Testbench

A **testbench** is a specialized Verilog module that creates a controlled environment to test and verify your design's functionality.

Testbench characteristics:

- Testbenches have no primary inputs or outputs - they generate everything internally.
- Creates and applies various input combinations to exercise the design.

![testbench](./Images/testbench.png)

## iverilog-Based Simulation Flow

The iverilog simulation process transforms Verilog code into visual waveform analysis through a systematic workflow.

### Simulation Process

1.Combine design and testbench files using iverilog compiler.

2.Create VCD files for signal analysis.

3.View results using GTKWave waveform viewer(Vim).

![Flow](./Images/iverilog_simulation_flow.png)

### Key Components

**VCD File (Value Change Dump)**: VCD files are industry-standard simulation dump files that record every signal transition during simulation.

**GTKWave** : GTKWave is a waveform viewer that transforms VCD data into interactive signal plots for design analysis and debugging.

## D Latch Execution
### Verilog Code
```bash
module good_latch (input clk, input reset,input d , output reg q);
always @(clk,reset,d)
begin
     if(reset)
        q<=1'b0;
     else if(clk)
        q<=d;
end
endmodule
```
### Testbench Code
```bash
`timescale lns/lps
module tb_good_latch;
     reg clk,reset,d;
     wire q;
     good_latch uut(
          .clk(clk),
          .reset(reset),
          .d(d),
          .q(q)
     );
     initial begin
     $dumpfile("tb_good_latch.vcd");
     $dumpvars(0,tb_good_latch);
     clk=0;
     reset=1;
     d=0;
     #330 $finish;
     end
always  #20 clk=~clk;
always #23 d=~d;
always #15 reset=0;
endmodule
```

## Environmental Setup
**1. Create a directory and clone it to 'sky130RTLDesignAndSynthesisWorkshop.git'**
```bash
cd VLSI
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```
**2. Get into design folders**
```bash
cd /home/meena/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files #
```
![directory](./Images/directory.png)

**3. Loading the latch into the simulator and generating vcd file**
```bash
iverilog good_latch.v tb_good_latch.v
./a.out
```
![vcdoutput](./Images/vcdoutput.png)

**4. Output Waveform command**
```bash
gtkwave tb_good_latch.vcd
```
>vcd file is dumped to gtkwave for visualising the output.
>
>The output of D latch can be through the waveform.

![waveform](./Images/waveform_latch.png)







