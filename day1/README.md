# VERILOG RTL DESIGN AND SYNTHESIS

- **Icarus Verilog** for simulation
- **GTKWave** for waveform viewing
- **Yosys** for RTL synthesis
- **ABC** (invoked via Yosys) for technology mapping / optimization

It includes example RTL, a testbench that generates a VCD, and a Yosys script for synthesis.

A testbench is a non-synthesizable Verilog code written to verify the behavior of a design (RTL module).
It does not become hardware — instead, it is used only for simulation.

First clone the Github repository of the Workshop
```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop
cd sky130RTLDesignAndSynthesisWorkshop
```
The repository contains the verilog codes and Testbenches for the Upcoming labs.
## MUX Design and Synthesis

### Workflow:

- Compile your design:
```bash
iverilog good_mux.v tb_good_mux.v
```
<img width="993" height="600" alt="Screenshot from 2025-09-27 21-35-43" src="https://github.com/user-attachments/assets/1bcee627-057b-487e-90da-6e3f338e3c6a" />
- Run the simulation:
```bash
./a.out
```

- Visualize signals using GTKWAVE:
```bash
gtkwave tb_good_mux.vcd
```
<img width="1215" height="424" alt="image" src="https://github.com/user-attachments/assets/66a4026c-4b7b-4461-abe6-c65daff2bb56" />


### Gate level Netlist Synthesis using YOSYS

- The `read_liberty` command is used to read a standard cell library file (`.lib`) into Yosys.

- The `read_verilog` command tells Yosys to read your RTL code written in Verilog and import it into its internal database.

- The `synth -top` command is used to synthesize a Verilog design and specify which module is the top-level module of your design hierarchy.

- The `abc -liberty` command is used to map your synthesized design to a gate-level netlist using ABC, a logic synthesis and verification tool, with a provided Liberty (`.lib`) file that defines the standard cell library.

- `write_verilog` is a command used to export your synthesized or processed design into a Verilog netlist

- `gvim` command is used of viewing and editing source code.

```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog good_mux.v
synth -top good_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr good_mux_netlist.v
!gvim good_mux_netlist.v


```
<img width="993" height="600" alt="image" src="https://github.com/user-attachments/assets/7a89fdf6-5e5e-490f-b153-59768e42f491" />


<img width="1202" height="652" alt="image" src="https://github.com/user-attachments/assets/ed4905e7-d3a2-4442-9894-10aeb8588e31" />
