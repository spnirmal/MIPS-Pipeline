# MIPS32 Pipelined Processor

This repository contains the Verilog implementation of a 5-stage pipelined MIPS32 processor, along with testbenches for simulation. The processor supports various arithmetic, logical, memory, and control instructions, enabling efficient instruction execution.

## Files in this Repository

- `mips32_pipeline.v` - Verilog implementation of the MIPS32 pipelined processor.
- `mips32_tb_ex1.v` - Testbench for verifying the functionality of the processor.
- Additional testbench files for different instruction sets.

## Features

- Implements a 5-stage pipeline: Fetch, Decode, Execute, Memory, Writeback.
- Supports various MIPS32 instructions.
- Simulation using Icarus Verilog (`iverilog`) and waveform analysis using GTKWave.

## Supported MIPS32 Instructions

### **1. Arithmetic & Logical Instructions**
| Instruction | Type  | Description |
|-------------|-------|-------------|
| `ADD rd, rs, rt` | R | rd = rs + rt |
| `ADDI rt, rs, imm` | I | rt = rs + imm |
| `SUB rd, rs, rt` | R | rd = rs - rt |
| `SUBI rt, rs, imm` | I | rt = rs - imm |
| `MUL rd, rs, rt` | R | rd = rs * rt |
| `AND rd, rs, rt` | R | rd = rs & rt |
| `OR rd, rs, rt` | R | rd = rs | rt |
| `SLTI rt, rs, imm` | I | Set rt = 1 if rs < imm |

### **2. Memory Instructions**
| Instruction | Type  | Description |
|-------------|-------|-------------|
| `LW rt, offset(rs)` | I | Load word from memory |
| `SW rt, offset(rs)` | I | Store word to memory |

### **3. Branch & Control Instructions**
| Instruction | Type  | Description |
|-------------|-------|-------------|
| `BNEQZ rs, offset` | I | Branch if rs != 0 |
| `BEQZ rs, offset` | I | Branch if rs == 0 |
| `HLT` | - | Halt processor execution |

## Simulation Instructions

1. **Run the simulation using Icarus Verilog:**
   ```sh
   iverilog -o mips32_sim mips32_pipeline.v mips32_tb_ex1.v
   vvp mips32_sim
   ```
2. **View waveforms in GTKWave:**
   ```sh
   gtkwave dump.vcd
   ```

## Future Improvements

- Implement forwarding and hazard handling for advanced scenarios.
- Add more test cases with complex instruction sequences.
- Instructions must be converted to hex and added to the testbench.
- Each instruction set currently requires a custom testbench; future versions may directly accept MIPS instructions.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

This README provides a concise overview of the project, supported instructions based on your Verilog source, and how to simulate it.

