# RISC-V Accelerator Integration Practice

## Project Objective
The primary objective of this project is to practice and document the system-level integration of custom hardware accelerators into a RISC-V SoC. The focus is on mastering the interfacing mechanism between the Rocket Chip and a custom DSP (Digital Signal Processing) unit using the RoCC (Rocket Custom Coprocessor) protocol.

## Key Focus Areas
- **Interface Protocol**: Mastering the handshake and data flow of the RoCC interface.
- **Instruction Extension**: Mapping high-level DSP functions to custom RISC-V opcodes (custom0–custom3).
- **System-on-Chip Integration**: Configuring the Rocket Chip generator to instantiate and connect custom co-processors.
- **Hardware–Software Co-design**: Validating the hardware integration by writing C-based drivers and test programs.

## Technical Specifications
- **Base SoC**: Rocket Chip Generator (RV64)
- **HDL**: Chisel (Scala-based hardware description)
- **Interface**: RoCC (tightly coupled coprocessor interface)
- **ISA Extension**: RISC-V custom instructions (custom0–custom3)
- **Verification**: Verilator simulation + C-based functional testing

## Project Structure
* `src/main/scala/`: Chisel source code for the DSP accelerator and RoCC wrapper.
* `src/test/scala/`: ScalaTest and ChiselTest files for RTL verification.
* `software/`: C-based test applications and header files for custom instructions.

## Integration Workflow

### 1. Interface Definition
Define a Chisel module that implements the RoCCIO bundle:
- `cmd` (instruction + rs1/rs2 operands)
- `resp` (rd writeback)
- `busy` (accelerator is processing)
- `interrupt` (optional)
- `mem` (TileLink memory access, optional)

This module wraps the DSP datapath and exposes it to the Rocket core.

### 2. Instruction Decoding
Custom instructions are mapped to DSP operations using `funct7`:

| Opcode  | funct7 | Operation     |
|---------|--------|---------------|
| custom0 | 0x01   | FIR filter    |
| custom0 | 0x02   | CIC decimator |
| custom0 | 0x03   | Vector MAC    |

The RoCC wrapper decodes the command and routes operands to the DSP unit.

### 3. Datapath Wiring
Two integration models are supported:
- **Register-based accelerator** (no memory access)
- **Memory-backed accelerator** (TileLink load/store)

This project uses the **register-based model** for simplicity.


### 4. SoC Configuration
Attach the accelerator to the Rocket core using a Config fragment:

```scala
class WithDSPAccelerator extends Config((site, here, up) => {
  case BuildRoCC => List(
    (p: Parameters) => new DSPAccelerator(OpcodeSet.custom0)(p)
  )
})

