# MIPS32 Pipelined Processor - Technical Documentation

This repository presents a Verilog-based implementation of a 5-stage pipelined MIPS32 processor. The design simulates real-world processor execution, modeling a realistic instruction flow using simulation tools like Icarus Verilog and GTKWave.

---

## 📁 Project Structure

- `mips32_pipeline.v` – Main Verilog file implementing the MIPS32 pipeline.
- `mips32_tb_ex1.v` – Primary testbench to simulate and verify processor behavior.

---

## 🔧 Architecture Overview

The MIPS32 processor implemented here follows the classic 5-stage pipeline architecture:

1. **Instruction Fetch (IF)**
2. **Instruction Decode/Register Fetch (ID)**
3. **Execution/Effective Address Calculation (EX)**
4. **Memory Access (MEM)**
5. **Write Back (WB)**

Each instruction progresses one stage per clock cycle, enabling simultaneous execution of multiple instructions and improving throughput.

## Pipelined vs Non-Pipelined MIPS 

| Feature                  | Non-Pipelined MIPS                | Pipelined MIPS                            |
|--------------------------|-----------------------------------|-------------------------------------------|
| **Execution**            | One instruction at a time         | Multiple instructions concurrently        |
| **CPI**                  | ~5 cycles per instruction         | ~1 cycle per instruction (after fill)     |
| **Performance**          | Slower                            | Faster throughput                         |
| **Complexity**           | Simpler hardware                  | Requires hazard handling                  |
| **Latency**              | Fixed                             | May have stalls (due to hazards)          |
| **Utilization**          | Idle hardware each cycle          | All pipeline stages active                |
| **Best for**             | Simulations, low-performance      | Real-world, high-performance CPUs         |

---
### non-pipelined
![non_pipelined](https://github.com/spnirmal/MIPS-Pipeline/blob/main/.assets/nonpilpelined.jpeg)

### pipelined
![pipelined](https://github.com/spnirmal/MIPS-Pipeline/blob/main/.assets/pipelined.jpeg)



The MIPS processor has 32 registers thus the name MIPS32here are all the 32 refisters in the processor with their uses.understanding this will allow us to write better mips code.
## 🧾 MIPS Register File

| Register No. | Name    | Usage Description                              |
|--------------|---------|------------------------------------------------|
| $0           | $zero   | Constant value 0 (hardwired)                   |
| $1           | $at     | Assembler temporary                            |
| $2–$3        | $v0–$v1 | Function return values                         |
| $4–$7        | $a0–$a3 | Arguments to functions                         |
| $8–$15       | $t0–$t7 | Temporary variables (caller-saved)             |
| $16–$23      | $s0–$s7 | Saved variables (callee-saved)                 |
| $24–$25      | $t8–$t9 | More temporary variables (caller-saved)        |
| $26–$27      | $k0–$k1 | Reserved for OS kernel                         |
| $28          | $gp     | Global pointer                                 |
| $29          | $sp     | Stack pointer                                  |
| $30          | $fp     | Frame pointer                                  |
| $31          | $ra     | Return address for function calls              |


---

## 🧠 How the Pipeline Works

### 🟩 1. **Instruction Fetch (IF)**
The processor fetches instructions from memory. If a branch is taken, PC is updated accordingly. Otherwise, it fetches the next sequential instruction.

```verilog
if (((EX_MEM_IR[31:26] == BEQZ) && (EX_MEM_cond == 1)) || 
    ((EX_MEM_IR[31:26] == BNEQZ) && (EX_MEM_cond == 0))) begin
    IF_ID_IR <= #2 Mem[EX_MEM_ALUOut];
    PC <= #2 EX_MEM_ALUOut + 1;
end else begin
    IF_ID_IR <= #2 Mem[PC];
    PC <= #2 PC + 1;
end
```

### 🟨 2. **Instruction Decode (ID)**
The instruction is decoded, register values are fetched, and immediate values are sign-extended.

```verilog
ID_EX_A <= Reg[IF_ID_IR[25:21]];  // rs
ID_EX_B <= Reg[IF_ID_IR[20:16]];  // rt
ID_EX_Imm <= {{16{IF_ID_IR[15]}}, IF_ID_IR[15:0]}; // Sign-extend
```

Opcode is used to determine the type of instruction:
```verilog
case (IF_ID_IR[31:26])
  ADD, SUB, AND, OR, SLT, MUL: ID_EX_type <= RR_ALU;
  ADDI, SUBI, SLTI:           ID_EX_type <= RM_ALU;
  LW:                         ID_EX_type <= LOAD;
  SW:                         ID_EX_type <= STORE;
  BNEQZ, BEQZ:                ID_EX_type <= BRANCH;
  HLT:                        ID_EX_type <= HALT;
endcase
```

### 🟧 3. **Execution (EX)**
ALU performs computations based on the decoded instruction type:

```verilog
case (ID_EX_type)
  RR_ALU:
    case (ID_EX_IR[31:26])
      ADD: EX_MEM_ALUout <= ID_EX_A + ID_EX_B;
      SUB: EX_MEM_ALUout <= ID_EX_A - ID_EX_B;
      MUL: EX_MEM_ALUout <= ID_EX_A * ID_EX_B;
      ...
    endcase

  RM_ALU:
    case (ID_EX_IR[31:26])
      ADDI: EX_MEM_ALUout <= ID_EX_A + ID_EX_Imm;
      ...
    endcase

  LOAD, STORE:
    EX_MEM_ALUout <= ID_EX_A + ID_EX_Imm;

  BRANCH:
    EX_MEM_ALUout <= ID_EX_NPC + ID_EX_Imm;
    EX_MEM_cond <= (ID_EX_A == 0);
endcase
```

### 🟦 4. **Memory Access (MEM)**
For LOAD and STORE instructions, memory is accessed:

```verilog
case (EX_MEM_type)
  LOAD:
    MEM_WB_LMD <= Mem[EX_MEM_ALUout];

  STORE:
    Mem[EX_MEM_ALUout] <= EX_MEM_B;
endcase
```

### 🟩 5. **Write Back (WB)**
The result is written back to the appropriate register.

```verilog
case (MEM_WB_type)
  RR_ALU:
    Reg[MEM_WB_IR[15:11]] <= MEM_WB_ALUout;
  RM_ALU:
    Reg[MEM_WB_IR[20:16]] <= MEM_WB_ALUout;
  LOAD:
    Reg[MEM_WB_IR[20:16]] <= MEM_WB_LMD;
  HALT:
    HALTED <= 1;
endcase
```

---

## ✅ Supported Instructions (as defined in `mips32_pipeline.v`)

### Arithmetic & Logical
| Instruction | Type | Opcode | Description |
|------------|------|--------|-------------|
| `ADD`  | R-type | 000000 | Register addition |
| `ADDI` | I-type | 000001 | Immediate addition |
| `SUB`  | R-type | 000010 | Register subtraction |
| `SUBI` | I-type | 000011 | Immediate subtraction |
| `MUL`  | R-type | 000100 | Multiplication |
| `AND`  | R-type | 000101 | Bitwise AND |
| `OR`   | R-type | 000110 | Bitwise OR |
| `SLTI` | I-type | 000111 | Set less than immediate |

### Memory Instructions
| Instruction | Type | Opcode | Description |
|-------------|------|--------|-------------|
| `LW`  | I-type | 001000 | Load word from memory |
| `SW`  | I-type | 001001 | Store word to memory |

### Branch & Control
| Instruction | Type | Opcode | Description |
|-------------|------|--------|-------------|
| `BEQZ` | I-type | 001010 | Branch if equal to zero |
| `BNEQZ` | I-type | 001011 | Branch if not equal to zero |
| `HLT`  | I-type | 111111 | Halt execution |

---
## 🧩 MIPS R-Type Instruction Format

R-type instructions are used for arithmetic and logical operations that involve register operands. The format of a 32-bit R-type instruction is as follows:

```
 31         26 25     21 20     16 15     11 10      6 5        0
+-------------+---------+---------+---------+---------+--------+
|   opcode    |   rs    |   rt    |   rd    | shamt   | funct  |
+-------------+---------+---------+---------+---------+--------+
```

- **opcode** (6 bits): Operation code (always `000000` for R-type)
- **rs** (5 bits): Source register 1
- **rt** (5 bits): Source register 2
- **rd** (5 bits): Destination register
- **shamt** (5 bits): Shift amount (used in shift instructions)
- **funct** (6 bits): Function code that determines the exact operation (e.g., `ADD`, `SUB`, `MUL`, etc.)

### Example: `ADD $t0, $t1, $t2`
- Adds contents of `$t1` and `$t2` and stores in `$t0`
- Binary Encoding:
  - `opcode`: 000000
  - `rs`: 01001 (for `$t1`)
  - `rt`: 01010 (for `$t2`)
  - `rd`: 01000 (for `$t0`)
  - `shamt`: 00000
  - `funct`: 100000 (ADD)

This format allows flexibility by using the `funct` field to differentiate instructions under the same `opcode`.

## 🧮 MIPS I-Type Instruction Format

```
 31         26 25     21 20     16 15                         0
+-------------+---------+---------+----------------------------+
|   opcode    |   rs    |   rt    |       immediate            |
+-------------+---------+---------+----------------------------+
```

- **opcode (6 bits)**: Specifies the operation (e.g., `ADDI`, `LW`, `SW`)
- **rs (5 bits)**: Source register
- **rt (5 bits)**: Destination register (or source in case of `SW`)
- **immediate (16 bits)**: Constant value or memory offset

### ✅ Example: `ADDI $t0, $t1, 10`
- Adds `10` to the value in `$t1` and stores the result in `$t0`
- Fields:
  - `opcode`: `000001`
  - `rs`: `$t1` (register 9)
  - `rt`: `$t0` (register 8)
  - `immediate`: `0000 0000 0000 1010`

---
## 🔪 Simulation

### Compiling and Running
```sh
git clone https://github.com/yourusername/mips32-pipeline.git
cd mips32-pipeline
iverilog -o sim mips32_pipeline.v mips32_tb_ex1.v
vvp sim
gtkwave dump.vcd
```

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

---

## 💡 Novelty & Use Cases
- Educational processor simulation for learning pipelining.
- Demonstrates how to implement instruction execution in a 5-stage model.
- Can be extended to handle forwarding, stalls, exceptions, and full instruction sets.

---

## 📈 Future Improvements
- Accept raw MIPS instructions as input rather than hex.
- Auto-convert MIPS instructions to machine code inside testbenches.
- Design a unified testbench to eliminate the need for separate files.
- Enable a complete instruction set implementation by updating parameter definitions.

---

## 📜 License
This project is licensed under the [MIT License](LICENSE).

---

This technical document elaborates how the pipeline processor operates and how each stage cooperates to execute instructions efficiently. Feel free to fork and improve!

