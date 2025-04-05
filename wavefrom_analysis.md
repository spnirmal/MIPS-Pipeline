## 📊 GTKWave Output Analysis - MIPS32 Pipeline Simulation

This section explains how to analyze the waveform output from GTKWave for the `mips32_tb_ex1` testbench simulation. this is alos upliaded as waveform_analysis.md in this same reposotary.

---


#### 🧩 General Waveform Layout
GTKWave lists signals on the left and plots their values over simulation time on the right. Let's break down what to look for when tracking instruction execution and register updates. here is the generated waveform in gtkwave.
![waveform](https://github.com/spnirmal/MIPS-Pipeline/blob/main/.assets/waveform.png)

---

### 📌 1. Instruction Register Tracking

**Signals:**  
- `IF_ID_IR`, `ID_EX_IR`, `EX_MEM_IR`, `MEM_WB_IR`

These represent the **instruction** at each pipeline stage:
- `IF_ID_IR`: Instruction Fetch (IF)
- `ID_EX_IR`: Instruction Decode (ID)
- `EX_MEM_IR`: Execution (EX)
- `MEM_WB_IR`: Write Back (WB)

#### 🔎 Example:
At ~60ns, `IF_ID_IR = 2801000A`, which represents:
```assembly
ADDI R1, R0, 10
```
As time progresses, this instruction moves through:
- `ID_EX_IR`
- `EX_MEM_IR`
- `MEM_WB_IR`

Each signal shows the pipeline advancing the instruction.

---

### 📌 2. Register Operand Values

**Signals:**  
- `ID_EX_A`, `ID_EX_B`, `ID_EX_Imm`

These hold operand data:
- `ID_EX_A`: value of RS
- `ID_EX_B`: value of RT
- `ID_EX_Imm`: immediate value (sign-extended)

#### 🔎 Example:
```assembly
ADDI R1, R0, 10
```
- `ID_EX_A = 00000000` (R0 is always 0)
- `ID_EX_Imm = 0000000A` (decimal 10)

For `ADDI R2, R0, 20`, `ID_EX_Imm = 00000014` (20 in hex).

---

### 📌 3. ALU Output (Execution Results)

**Signal:**  
- `EX_MEM_ALUOut`

This shows results of ALU operations.

#### 🔎 Examples:
- `ADDI R1, R0, 10` → `EX_MEM_ALUOut = 0000000A`
- `ADDI R2, R0, 20` → `EX_MEM_ALUOut = 00000014`
- `ADD R4, R1, R2` → `EX_MEM_ALUOut = 0000001E`

You can confirm the ALU executed `10 + 20 = 30`.

---

### 📌 4. Writeback Confirmation

**Signals:**
- `MEM_WB_ALUOut`
- `MEM_WB_IR`

Once a value reaches `MEM_WB_ALUOut`, it's being written back to the register.

#### 🔎 Example:
If `MEM_WB_ALUOut = 00000014`, it means register R2 is being written with value 20.

---

### 📌 5. Program Counter (PC) Tracking

**Signal:**  
- `PC`

This tracks which instruction is being fetched. It increments every cycle unless modified by a branch.

---

### 📌 6. Register File Monitoring

You won’t directly see `Reg[0]`, `Reg[1]`, etc., unless exposed manually.

#### ✅ To view specific registers:
Modify your testbench:
```verilog
$dumpvars(0, cpu.Reg[1], cpu.Reg[2], cpu.Reg[3], cpu.Reg[4], cpu.Reg[5]);
```

Or add a monitor:
```verilog
$monitor("R1=%h R2=%h R3=%h R4=%h R5=%h", cpu.Reg[1], cpu.Reg[2], ...);
```
![vscode_output](https://github.com/spnirmal/MIPS-Pipeline/blob/main/.assets/output_vscode.png)
---

### 🧠 Summary Table

| What You Want To See        | Signal Name          | Example Values |
|-----------------------------|----------------------|----------------|
| Instruction at a stage      | `*_IR`               | 2801000A       |
| Source register values      | `ID_EX_A`, `ID_EX_B` | 00000000, ...  |
| Immediate value             | `ID_EX_Imm`          | 0000000A       |
| ALU computation result      | `EX_MEM_ALUOut`      | 0000001E       |
| Final result to write back  | `MEM_WB_ALUOut`      | 00000014       |
| Program counter             | `PC`                 | 00000001+      |

---
