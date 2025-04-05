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

## 🔪 Simulation

### Compiling and Running
```sh
git clone https://github.com/yourusername/mips32-pipeline.git
cd mips32-pipeline
iverilog -o sim mips32_pipeline.v mips32_tb_ex1.v
vvp sim
gtkwave dump.vcd
```

### GTKWave Analysis
- Observe register writes and ALU outputs.
- Ensure each stage performs as expected.
- Register values like Reg[0] to Reg[5] will reflect ADD, ADDI, etc.

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

