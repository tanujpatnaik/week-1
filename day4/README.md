# Gate Level Simulattion

Simulating the synthesized netlist (logic gates, flip-flops, interconnects) instead of the original RTL (behavioral code).

- The netlist is typically generated after logic synthesis (Verilog → standard cells).

- Simulation is done with the same testbench used for RTL but on the gate-level design.

- In GLS, the design is described in terms of AND, OR, NAND, DFFs, etc. instead of high-level RTL constructs.

Even if RTL simulation passes, synthesis can introduce changes. GLS is used to verify correctness after synthesis.

- Verify Netlist Functionality

   - Ensure the synthesized circuit matches RTL behavior.
   - Catch mismatches due to synthesis optimizations, constraints, or coding style issues.
- Check Timing (with SDF back-annotation)

  - At RTL, everything is ideal (no delays).
  - GLS with Standard Delay Format (SDF) includes real gate and wire delays.
  - Helps catch:
     - Setup/Hold time violations.
     - Race conditions.
     - Glitches/Hazards.

- Reset/Initialization Verification

  - At gate-level, flip-flops may not have defined startup values (unlike RTL where initial blocks or defaults can mask issues).
  - GLS ensures design initializes correctly with reset.
- Low-Power & Scan Insertion Checks
   - After scan-chain insertion (DFT) or clock-gating transformations, GLS checks that the new logic still works.
 

## Functional Gate-Level Simulation

- Simulates the gate-level netlist but assumes ideal delays (0-delay or unit-delay).

- Purpose:

  - Check functional correctness of the netlist after synthesis.

  - Verify that the synthesis process didn’t change the design’s intended behavior.
- Finds:
   - Issues from synthesis (e.g., latch inference, unconnected signals).
   - Problems with resets and initialization.

- It does not check:
  - Real timing behavior (setup/hold, race conditions, glitches).

- Fast compared to timing-aware GLS.
- Still much slower than RTL sim.

## Timing-Aware Gate-Level Simulation

Simulates the gate-level netlist with real timing delays back-annotated from SDF (Standard Delay Format) file.

- Purpose:

  - Verify the true timing behavior of the design.

  - Check that setup, hold, recovery, removal, and propagation delays are met.

- Finds:

  - Setup/hold time violations.

  - Race conditions.

  - Glitches and hazards due to real gate/wire delays.

  - Clock-domain crossing (CDC) metastability issues.

- Very accurate — close to silicon behavior.
- Extremely slow — used sparingly (e.g., final signoff).

### Example

#### Ternary operator


<img width="607" height="132" alt="ternary operator code" src="https://github.com/user-attachments/assets/4ba49ae9-0592-4c2c-9af6-1f525f6a7bec" />


- Simulate and observe the waveform of th RTL code using iverilog and GTKWave.


```bash
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

<img width="1212" height="344" alt="image" src="https://github.com/user-attachments/assets/0f7a1f55-ef82-43e5-9a8b-eb5fdf13837e" />




- Generate the netlist file and observe the output waveforms using iverilog and GTKWave.

  ```bash
  read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  read_verilog ternary_operator_mux.v
  synth -top ternary_operator_mux
  abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  write_verilog ternary_operator_mux_net.v
  show ternary_operator_mux_net
  ```



<img width="1212" height="659" alt="image" src="https://github.com/user-attachments/assets/2585c480-16c1-4c54-8533-72f3ae6349ec" />
```bash
  iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
  ./a.out
  gtkwave tb_ternary_operator_mux.vcd
  ```

<img width="1209" height="215" alt="gls netlist gtkwave of ternary operator" src="https://github.com/user-attachments/assets/7fe5caef-cb8e-41c6-a440-3297b8f3267d" />

##### 🔎 Observation

- By comparing both the output waveforms we can observe that the waveforms are identical.
- Matching RTL and gate-level waveforms means functional correctness is confirmed.

# Synthesis and Simulation Mismatches


A synthesis–simulation mismatch (SSM) happens when:

- Your RTL simulation (pre-synthesis, behavioral) produces one result.

- But the post-synthesis (gate-level) simulation produces a different result.

- This means the RTL you wrote did not map to hardware the way you expected.

- Reasons for Synthesis–Simulation Mismatches:

## Incorrect RTL Coding  
Using constructs that simulate fine but are not synthesizable.

## Missing Sensitivity List
If sensitivity list doesn’t include all inputs, then this mismatch occurs.

### Bad_mux


<img width="554" height="219" alt="bad mux code" src="https://github.com/user-attachments/assets/29d06100-c646-45aa-8fef-8bd95a3e4bc4" />

- We can see from the code that the always block sensitivity list doesn't contain th inputs i1 and i0.

- We first simulate and observe the waveform of th RTL code using iverilog and GTKWave.

```bash
iverilog bad_mux.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```

<img width="1201" height="299" alt="image" src="https://github.com/user-attachments/assets/8e44c84b-3220-4b54-b977-5e5e098f2dbb" />



- Then we generate the netlist file and observe the output waveforms using iverilog and GTKWave.

  ```bash
  read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  read_verilog bad_mux.v
  synth -top bad_mux
  abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  write_verilog bad_mux_net.v
  show bad_mux_net
  ```

  <img width="611" height="215" alt="bad mux output" src="https://github.com/user-attachments/assets/aedea68e-bdf7-4a94-9858-c1bbadd390d7" />


  ```bash
  iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux_net.v tb_bad_mux.v
  ./a.out
  gtkwave tb_bad_mux.vcd
  ```

<img width="1207" height="215" alt="gtkwave of netlist bad mux" src="https://github.com/user-attachments/assets/b6d7ea88-b64a-4021-8c46-5023d149c27b" />

- Now we have to compare both waveforms.

#### 🔎 Observation
- The RTL coding simulation has the functionality of a double edge sensitive flip flop in which the input select line `sel` acts as the clock.
- The Generated Netlist has the functionality of a 2:1 MUX.
- This mismatch is occured due to the missing sensitivity list.

## Blocking (=) vs Non-Blocking (<=) Assignments

Using the wrong type in sequential logic can cause mismatches.

- Blocking Assignment
   - Executes statement by statement in order, blocking the next statement until it finishes.
   - Usage:
     - Usually in combinational logic (`always @(*)`)
     - Not recommended in sequential (`always @(posedge clk)`) because it can cause race conditions.
- Non-Blocking Assignment
  - Evaluates the right-hand side immediately, but updates the left-hand side at the end of the time step.
  - Usage:
     - Usually in sequential logic (`always @(posedge clk)`)
     - Ensures all registers are updated in parallel, like real hardware flip-flops.

### Blocking Caveat

<img width="881" height="157" alt="blocking caveat verilog code" src="https://github.com/user-attachments/assets/7f0d3de6-7e58-43bf-9391-80b2f883b9d1" />


```bash
iverilog blockig_caveat.v tb_blockig_caveat.v
./a.out
gtkwave tb_blockig_caveat.vcd
```

<img width="1203" height="215" alt="gtkwave of rtl blocking caveat" src="https://github.com/user-attachments/assets/66448560-14cc-432a-a7a7-3d4c79c11ae1" />


```bash
  read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  read_verilog blocking_caveat.v
  synth -top blocking_caveat
  abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  write_verilog blocking_caveat_net.v
  show blocking_caveat_net
  ```

<img width="598" height="215" alt="blocking caveat output" src="https://github.com/user-attachments/assets/7efc1a97-7f0d-4be4-adfb-88aa0d90a4c1" />


 ```bash
  iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat_net.v tb_blocking_caveat.v
  ./a.out
  gtkwave tb_blocking_caveat.vcd
  ```

<img width="1208" height="215" alt="gtkwave of netlist blocking caveat" src="https://github.com/user-attachments/assets/38058886-cdad-4ede-9c0b-cab12b34f857" />


#### Observation
- RTL simulation executes statements top-down, so you see one behavior.
- Synthesis maps these to flip-flops, which are parallel hardware. The hardware updates simultaneously
- propagate incorrect data through registers → causes wrong outputs in downstream logic.
- This mismatch is occured due to the Blocking Caveat.
