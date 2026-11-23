# RV32I Processor: Single-Cycle Implementation

This repository contains a modular SystemVerilog implementation of a 32-bit, single-cycle processor for the RV32I base integer instruction set. Its module boundaries and directory names follow the course's [RISC-V Single-Cycle reference repository](https://github.com/pranav0x2212/RISC-V-Single-Cycle), so the datapath can be reviewed stage by stage.

## What is included

The processor is divided into instruction fetch, decode, execute, and writeback. Decode is further split into instruction-field extraction, immediate construction, control decoding, and register storage. Execute combines the ALU, ALU-control decoder, operand selection, and control-flow target calculation. The top-level integration module is `single_cycle_riscv`.

Supported operations are the RV32I upper-immediate operations (`LUI`, `AUIPC`), jumps (`JAL`, `JALR`), six conditional branches, byte/halfword/word loads and stores, and the base integer register-register and immediate arithmetic, logical, comparison, and shift operations. The M, C, floating-point, CSR, and privileged extensions are outside this implementation's scope.

## Files

- `IF_Stage/rtl.sv` — PC state, sequential address, and next-PC selection
- `Decoder/rtl.sv` — opcode-dependent instruction field extraction
- `Immediate_Generator/rtl.sv` — sign extension and immediate formatting
- `Register_File/rtl.sv` — two combinational read ports and one clocked write port
- `Main_Control/rtl.sv` — instruction-class control outputs
- `ID_Stage/rtl.sv` — decode-stage module connections
- `ALU_Control/rtl.sv` and `ALU/rtl.sv` — ALU operation selection and execution
- `EX_Stage/rtl.sv` — operands, branch decision, and jump/branch target
- `WB_Stage/rtl.sv` — register writeback selection
- `Single_Cycle_RISCV/rtl.sv` — full processor integration
- `rnr-stage.yaml` — course-style module/source manifest

## Core connections

The top-level interface is:

```text
Inputs:  clk, reset, stall, instruction[31:0], mem_out[31:0]
Outputs: rs2_data[31:0], alu_out[31:0], mem_write, mem_read,
         fn3[2:0], opcode[6:0], address[31:0]
```

Reset is active high and asynchronous. It initializes `address` to `32'h8000_0000`. With `stall` asserted, the fetch address remains unchanged; register-file writes and memory read/write requests are disabled.

## Memory model expected by the core

Both memories are external to the processor. The instruction memory uses `address` for a combinational read and returns its 32-bit word on `instruction`. The data-memory side uses `alu_out` as the byte address. It receives store data on `rs2_data`, access type on `fn3`, and request controls on `mem_read` and `mem_write`.

Before a load value is presented at `mem_out`, the external data-memory logic must select the addressed byte or halfword and extend it to 32 bits: signed extension for `LB` and `LH`, zero extension for `LBU` and `LHU`. `LW` returns the addressed word. For stores, the external logic uses the address and `fn3` to route the low byte, low halfword, or full word into the appropriate memory lanes and applies the corresponding write enables.

This interface assumes the instruction and load data are available combinationally during the processor cycle. No memory array or testbench is included in this source bundle.

## Build-file list

`rnr-stage.yaml` identifies `single_cycle_riscv` as the integration module and includes the RTL source for each module in its hierarchy. Add the listed SystemVerilog files to the course flow or to a simulator project as a single design.
