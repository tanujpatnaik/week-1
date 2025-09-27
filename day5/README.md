# 📌 Optimisation in Synthesis

The process of refining the RTL during logic synthesis to produce a netlist that meets design goals:

- Area → fewer gates, smaller silicon

- Timing → faster critical paths, higher clock frequency

- Power → lower dynamic and static power

All while maintaining functional correctness.

## 🔹 `if-else` Construct

Characteristics:

- Priority encoding
   - Evaluates conditions top-down.
   - The first true condition is selected; others are ignored.

- Good for:
   - Conditional checks with priority.

   - When some conditions are mutually exclusive.

- Hardware mapping

   - Synthesizer maps if-else to multiplexers or priority encoders.

   - Can result in slightly deeper logic if multiple priority checks exist.

- Optimization tip

   - Common logic in branches can be factored to reduce gate count.

## 🔹 `case` Construct

Characteristics:

- Equality-based selection

   - No priority; output depends only on matching value.

- Good for:

   - Multiplexers, FSMs, table-driven logic.
  
   - When all input values are mutually exclusive.

- Hardware mapping

   - Synthesizer maps case to multiplexers or decoder + OR/AND logic.

   - Easier to optimize if branches share logic.

- Optimization tip

   - Merge duplicate branches, remove unreachable states, and exploit don’t-care values (casex, casez).

## ⚠️ Errors in `if-else` Constructs
### Incomplete assignment (latch inference)

If a signal is not assigned in all branches, synthesis may infer a latch.

### Examples
### 1.incomp_if

<img width="970" height="250" alt="incomplete if verilog code" src="https://github.com/user-attachments/assets/b86be870-620b-43b0-818a-dea6957178c6" />

- The verilog code contains a incomplete if-statement.

- Observe the RTL design waveform in GTKWave.

```bash
iverilog incomp_if.v tb_incomp_if.v
./a.out
gtkwave tb_incomp_if.vcd
```

<img width="1211" height="250" alt="gtkwave of incomplete if rtl" src="https://github.com/user-attachments/assets/dfdbc345-271c-40ae-8a60-0e2a96a5a4f4" />

```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog incomp_if_net.v
show
```


<img width="601" height="302" alt="incomplete if output" src="https://github.com/user-attachments/assets/01cf9d80-3ff4-4f52-8af6-e65e113f18d4" />


```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v incomp_if_net.v tb_incomp_if.v
./a.out
gtkwave tb_incomp_if.vcd
```


<img width="1204" height="178" alt="gtkwave of incomplete if netlist" src="https://github.com/user-attachments/assets/12bc10ad-00a7-4f13-8048-cc420ee66138" />


#### 🔎 Observation

- We can observe from both the waveforms that the output does not depend on the input i2.
- A latch has been inferred in the generated Netlist.
- This increase unnecessary hardware instantiation.


### incomp_if2

<img width="628" height="212" alt="incomplete if2 verilog code" src="https://github.com/user-attachments/assets/c389a1af-3a10-4c69-9b7b-58afff763a35" />



```bash
iverilog incomp_if.v tb_incomp_if.v
./a.out
gtkwave tb_incomp_if.vcd
```


<img width="1204" height="231" alt="gtkwave of incomplete if2 rtl" src="https://github.com/user-attachments/assets/7a6adf60-b514-464f-a183-8c988e8a3636" />


```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if2.v
synth -top incomp_if2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog incomp_if2_net.v
show
```

<img width="1204" height="290" alt="incomplete if2 output" src="https://github.com/user-attachments/assets/2de4e26e-8aef-48b1-b153-1251ce03c5a8" />


```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v incomp_if2_net.v tb_incomp_if2.v
./a.out
gtkwave tb_incomp_if2.vcd
```


<img width="1204" height="290" alt="gtkwave of incomplete if 2 netlist" src="https://github.com/user-attachments/assets/038142e4-0ff5-408b-a50e-2012b2cb1982" />



#### 🔎 Observation

- A latch has been inferred in the generated Netlist along with the MUX.
- This increase unnecessary hardware instantiation and incorrect functionality.

### Blocking assignment in sequential logic

- Using = inside always @(posedge clk) can cause race conditions.

- Fix: Use non-blocking assignments (<=) for sequential logic.


## ⚠️ Errors in `case` constructs

### Incomplete Case → Latch Inference and No Default Statement

- If not all possible values of the case expression are covered, the synthesizer may infer a latch to hold the value.

- If no default is given, synthesis may assume don’t-care or infer a latch.


- ❌ Risk of mismatch.

- ✅ Fix: cover all cases or add a default.

<img width="707" height="174" alt="incomplete case verilog code" src="https://github.com/user-attachments/assets/f1f4dd64-a557-4b64-9878-9a3097199255" />



```bash
iverilog incomp_case.v tb_incomp_case.v
./a.out
gtkwave tb_incomp_case.vcd
```


<img width="1201" height="212" alt="gtkwave of incomplete case rtl" src="https://github.com/user-attachments/assets/d5000c72-460f-46ff-9a25-d08a3ebd3a59" />


```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_case.v
synth -top incomp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog incomp_case_net.v
show
```


<img width="1201" height="230" alt="incomplete case output" src="https://github.com/user-attachments/assets/644715c7-6acf-45a8-b59b-75dba79a53ef" />


```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v incomp_case_net.v tb_incomp_case.v
./a.out
gtkwave tb_incomp_case.vcd
```

<img width="1216" height="247" alt="gtkwave of incomplete case netlist" src="https://github.com/user-attachments/assets/6ef1b60a-5075-4c26-9268-37cba1bf5eb8" />

#### 🔎 Observation

- We can observe that a D-latch has been inferred in the Generated Netlist.
- We have intended to implement a MUX but in the output also has been generated.

### Priority vs Parallel Confusion

- case is parallel evaluation, unlike if-else (priority).

- But casex and casez can introduce unintended priority because x and z match multiple patterns.

- ❌ Lower branches can get shadowed.


<img width="656" height="262" alt="bad case verilog code" src="https://github.com/user-attachments/assets/31b59754-2d7e-4eda-9ada-b104ae08ae7d" />


```bash
iverilog bad_case.v tb_bad_case.v
./a.out
gtkwave tb_bad_case.vcd
```


<img width="1208" height="247" alt="gtkwave of bad case rtl" src="https://github.com/user-attachments/assets/ad018d6f-12d4-48eb-9dc5-db9be90ca89f" />


```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_case.v
synth -top bad_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog bad_case_net.v
show
```



<img width="606" height="303" alt="bad case output" src="https://github.com/user-attachments/assets/2d347e15-1baa-4037-9b92-33c9faa06319" />


```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_case_net.v tb_bad_case.v
./a.out
gtkwave tb_bad_case.vcd
```


<img width="1180" height="233" alt="gtkwave of bad case netlist" src="https://github.com/user-attachments/assets/b1c9d4c5-f44f-4ebb-b6a7-ee752e5c86da" />


#### 🔎 Observation

- We can observe that the waveforms of the RTL design and Synthesis does not match when `sel=11`. 
- This is due to the parallel confusion.

### Partial-case-assignment

- A partial case assignment happens when not all output signals are assigned in every branch of a case statement.

- This can lead to unintended latch inference or mismatches between simulation and synthesis.

<img width="869" height="301" alt="partial assign case verilog code" src="https://github.com/user-attachments/assets/becc556c-465a-449b-aff5-cfbd7dcbf4ff" />

-As we can see `x` is not assigned when `sel=01`.


```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog partial_case_assign.v
synth -top partial_case_assign
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog partial_case_assign_net.v
show
```

<img width="1208" height="337" alt="partial case assign output" src="https://github.com/user-attachments/assets/cb376cca-ea97-4c3a-9705-2aa836a275f1" />


#### 🔎 Observation

- We can observe that a D-Latch has been inferred in the path of output `x`
- This latch is generated to preserve the value of `x` until the value of `sel` changes.



## ♻️ Looping Constructs(`for` loop and `for generate`)

### 🔹 `for` Loop

- Used inside procedural blocks (always, initial, task, function).

- Executes at runtime during simulation.

- Synthesizers “unroll” the loop (if the bounds are constant).

- ✅ Synthesizer will replicate the increment logic for all array elements.
  
- ❌ Cannot use for outside of a procedural block.

#### MUX using `for` loop

<img width="652" height="247" alt="mux for loop verilog code" src="https://github.com/user-attachments/assets/cde167ce-0f9f-4db2-8938-f3ab3fe0e912" />

- From the above code we can see that an integer variable `k` is used in the `for` loop.
- This integer `k` is used to specify the number of iterations of the loop.

```bash
iverilog mux_generate.v tb_mux_generate.v
./a.out
gtkwave tb_mux_generate.vcd
```

<img width="1180" height="282" alt="gtkwave of mux for loop rtl" src="https://github.com/user-attachments/assets/5f93810e-69dc-4603-8d39-78861fe80137" />


```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mux_generate.v
synth -top mux_generate
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog mux_generate_net.v
show
```


<img width="844" height="400" alt="mux for loop output" src="https://github.com/user-attachments/assets/e5573bf2-d4e4-49db-88de-51bd66d5d68d" />


```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v mux_generate_net.v tb_mux_generate.v
./a.out
gtkwave tb_mux_generate.vcd
```


<img width="1210" height="206" alt="gtkwave of mux for loop netlist" src="https://github.com/user-attachments/assets/0924cd9e-b727-4f82-8ed5-ca299dff661e" />





#### DE_MUX using `for` loop


<img width="1210" height="656" alt="demux case and for loop verilog codes" src="https://github.com/user-attachments/assets/dec725a3-f3a2-4c9d-82c5-a32248e453ea" />


```bash
iverilog mux_generate.v tb_mux_generate.v
./a.out
gtkwave tb_mux_generate.vcd
```

<img width="1210" height="378" alt="gtkwave of demux case rtl" src="https://github.com/user-attachments/assets/e93efac6-dbbf-4c7f-9bbe-b2ef2352a074" />


```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mux_generate.v
synth -top mux_generate
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog mux_generate_net.v
```




```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v mux_generate_net.v tb_mux_generate.v
./a.out
gtkwave tb_mux_generate.vcd
```

<img width="1210" height="378" alt="gtkwave of demux for loop netlist" src="https://github.com/user-attachments/assets/9dbc8482-19b7-451a-9f10-b0d1f528b683" />



#### 🔎 Observation

- We can observe that a 4:1 MUX and 1:4 DE-MUX is generated by using `for` loop.
- `for` is useful when we have to evaluate large number of expressions such as 256:1 MUX,1:256 DE-MUX.



### 🔹for-generate (Generate Loop)

- Used at elaboration/compile time (before simulation starts).

- Creates multiple instances of hardware modules/blocks.

- Requires a `genvar` index.

- Very useful for replicating structures (like adders, multiplexers, registers).

- ❌ Cannot be placed inside procedural blocks.

#### Ripple Carry Adder using `for generate`

<img width="660" height="413" alt="rca and fa verilog code" src="https://github.com/user-attachments/assets/7ce98e16-d6ea-4b4d-95e1-46bcec429aca" />


- From the above code we can see that a `genvar` index `i` is used.
- This `genvar` is used to specify the number of hardware replications required.

```bash
iverilog fa.v rca.v tb_rca.v
./a.out
gtkwave tb_rca.vcd
```

<img width="1210" height="161" alt="gtkwave of rca rtl" src="https://github.com/user-attachments/assets/f834aa9b-e853-4bd2-8233-3ab6c7b75754" />


```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog rca.v fa.v
synth -top rca
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog rca_net.v
show
```

<img width="816" height="683" alt="rca output" src="https://github.com/user-attachments/assets/6535456b-2be0-4fc8-b56a-261def06d310" />



```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v rca_net.v tb_rca.v
./a.out
gtkwave tb_rca.vcd
```


<img width="1199" height="238" alt="gtkwave of rca netlist" src="https://github.com/user-attachments/assets/6869a5d6-8ea4-4b66-ac0b-65d7f71840f4" />


#### 🔎 Observation

- Structural Clarity 🏗️

   - Without for-generate, you’d have to instantiate each full adder manually.

   - With for-generate, the loop replicates the hardware at compile/elaboration time, keeping the code short and scalable.

- Scalability 📏

   - By changing the parameter i, you can generate an adder of any bit-width without rewriting code.

   - E.g., i=4 → 4-bit adder, i=32 → 32-bit adder.

- Unique Naming 🏷️

   - Each full adder instance is uniquely named as GEN_fa[0].fa, GEN_fa[1].fa, etc.

   - This makes debugging in simulation easier.

- Hardware Expansion ⚡

   - Synthesizer expands the for-generate into N separate full adder instances.

   - The result is identical to manual instantiation, but less error-prone.
