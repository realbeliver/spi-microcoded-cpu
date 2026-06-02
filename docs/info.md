# SPI Microcoded CPU Design Notes

## Overview

This project implements a TinyTapeout-targeted 4-bit microcoded CPU that fetches its program from external SPI RAM.

In the current demo configuration:

- `ui_in[7:4]` carries operand `A`
- `ui_in[3:0]` carries operand `B`
- `uo_out[7:0]` returns the 8-bit product `A x B`

The design is intentionally small and easy to inspect. It is aimed at experimenting with external program storage, microcoded execution, and compact datapath control in a silicon-friendly TinyTapeout setting.

## Top-level architecture

The design is divided into three logical blocks:

1. `tt_um_spi_cpu_top`
2. `spi_wrap`
3. `ExecutionUnit`

### `tt_um_spi_cpu_top`

This is the TinyTapeout integration wrapper.

Responsibilities:

- Accepts the two 4-bit operands on `ui_in`
- Drives the 8-bit result on `uo_out`
- Connects the SPI-related interface to `uio_*`
- Instantiates and wires the internal CPU logic

### `spi_wrap`

This block manages instruction fetch and sequencing.

Responsibilities:

- Maintains the program counter
- Controls the SPI byte-read process
- Stores or forwards the fetched instruction byte
- Sequences execution of the lower nibble first and then the upper nibble
- Advances to the next program byte after both micro-operations are executed

### `ExecutionUnit`

This block performs the actual micro-operation execution.

At a high level, it contains:

- A register
- B register
- O register
- 8-bit accumulator
- Shift register
- Status/flag logic
- ALU/control decode logic

The execution unit receives a 4-bit micro-operation from `spi_wrap` and updates internal state accordingly.

## Program fetch model

The CPU uses an **external SPI memory** as program storage.

A fetched SPI byte contains two 4-bit micro-operations:

- `instr[3:0]` = first micro-operation to execute
- `instr[7:4]` = second micro-operation to execute

Execution order is:

1. Fetch one instruction byte from SPI memory
2. Execute lower nibble
3. Execute upper nibble
4. Increment program counter
5. Fetch next byte

This approach reduces the program-memory width while keeping the internal execution logic simple.

## Suggested fetch sequence

The fetch/control flow can be documented as:

1. Assert SPI transaction for the current program counter address
2. Read one byte from external memory
3. Latch fetched byte into an instruction register
4. Decode and execute lower nibble
5. Decode and execute upper nibble
6. Increment program counter
7. Repeat until the programmed routine completes

## External interface

> Replace placeholder entries with the exact RTL signal mapping used in your design.

### Operand and result interface

| Signal | Width | Direction | Description |
|---|---:|---|---|
| `ui_in[7:4]` | 4 | Input | Operand A |
| `ui_in[3:0]` | 4 | Input | Operand B |
| `uo_out[7:0]` | 8 | Output | Product/result output |

### SPI-facing interface

| Signal | Width | Direction | Description |
|---|---:|---|---|
| `uio_in[...]` | TBD | Input | SPI-related input path from external memory |
| `uio_out[...]` | TBD | Output | SPI-related output path to external memory |
| `uio_oe[...]` | TBD | Output | Output-enable control for bidirectional user IO |

If the design uses dedicated assignments such as CS, SCLK, MOSI, and MISO, replace the generic rows above with exact signal names and bit positions.

## Internal state elements

> Fill the reset values and exact usage from the RTL.

| Element | Width | Purpose | Reset value |
|---|---:|---|---|
| `A` | 4 | Operand/data register | TBD |
| `B` | 4 | Operand/data register | TBD |
| `O` | 8 or TBD | Output/result register | TBD |
| `ACC` | 8 | Accumulator for arithmetic flow | TBD |
| `SHIFT` | TBD | Shift/serial helper register | TBD |
| `FLAGS` | TBD | Status/condition bits | TBD |
| `PC` | TBD | Program counter for SPI byte fetch | TBD |

## Micro-instruction format

Each instruction byte stores two micro-operations. Each micro-operation is 4 bits wide.

### Packed byte format

```text
+---------+---------+
| [7:4]   | [3:0]   |
| op_hi   | op_lo   |
+---------+---------+
```

Execution order:

- Execute `op_lo`
- Execute `op_hi`

## Opcode table

> Replace this placeholder table with the exact opcode mapping implemented in `ExecutionUnit`.

| Opcode | Mnemonic | Operation | State updated | Flags affected |
|---|---|---|---|---|
| `0000` | `NOP` | No operation | None | None |
| `0001` | `TBD` | TBD | TBD | TBD |
| `0010` | `TBD` | TBD | TBD | TBD |
| `0011` | `TBD` | TBD | TBD | TBD |
| `0100` | `TBD` | TBD | TBD | TBD |
| `0101` | `TBD` | TBD | TBD | TBD |
| `0110` | `TBD` | TBD | TBD | TBD |
| `0111` | `TBD` | TBD | TBD | TBD |
| `1000` | `TBD` | TBD | TBD | TBD |
| `1001` | `TBD` | TBD | TBD | TBD |
| `1010` | `TBD` | TBD | TBD | TBD |
| `1011` | `TBD` | TBD | TBD | TBD |
| `1100` | `TBD` | TBD | TBD | TBD |
| `1101` | `TBD` | TBD | TBD | TBD |
| `1110` | `TBD` | TBD | TBD | TBD |
| `1111` | `TBD` | TBD | TBD | TBD |

This table is the most important missing technical reference in the current documentation. Without it, the microcode format is not fully specified.

## Reset and startup behavior

> Fill these items from RTL and simulation results.

Document the following explicitly:

- Reset polarity
- Which registers are cleared on reset
- Initial program counter value
- First SPI address accessed after reset
- Whether output stays zero until computation completes
- What defines the end of the multiplication sequence
- What happens if SPI data is missing or invalid

Recommended format:

| Item | Value |
|---|---|
| Reset polarity | TBD |
| Initial `PC` | TBD |
| Initial output value | TBD |
| First fetch address | TBD |
| Stop condition | TBD |

## Execution walkthrough

This section should explain how the multiplication demo progresses.

### Example: `A = 3`, `B = 5`

Inputs:

- `A = 4'b0011`
- `B = 4'b0101`

Expected output:

- `A x B = 15`
- `uo_out = 8'b00001111`

Document the routine step by step:

| Step | Program byte / nibble | Micro-operation | Internal effect | Notes |
|---|---|---|---|---|
| 1 | TBD | TBD | TBD | Load/setup |
| 2 | TBD | TBD | TBD | Shift/add or partial-product handling |
| 3 | TBD | TBD | TBD | Continue multiply sequence |
| 4 | TBD | TBD | TBD | Final accumulation |
| 5 | TBD | TBD | TBD | Write result to output register |

Replace the `TBD` entries with the actual microcode trace from simulation or from the ROM/RAM program image.

## Verification

The repository includes a `test/` directory and should summarize the verification approach here.

Recommended contents:

- Testbench structure
- How SPI memory is modeled
- How operands are applied
- How result checking is performed
- Whether all 16 x 16 input combinations are tested
- Whether reset timing and fetch timing are checked

Suggested table format:

| Test | Description | Expected result |
|---|---|---|
| Reset test | Apply reset and observe startup state | Registers and PC initialize correctly |
| Single multiply test | Apply one operand pair | `uo_out` matches expected product |
| Multiple operand sweep | Check all 4-bit combinations | All products match reference model |
| SPI fetch sequencing | Observe byte fetch and nibble order | Lower nibble executes before upper nibble |

## Design notes

This architecture trades internal memory for a simple external SPI program source. That makes the core smaller and keeps the instruction stream visible and easy to modify from outside the chip.

The main design trade-off is that execution correctness depends on the external memory interface and on a clearly documented micro-instruction encoding. For that reason, exact opcode definitions and SPI timing should be treated as part of the design specification, not just implementation details.

## Future improvements

Possible next steps include:

- Publish the final opcode map
- Add exact SPI timing diagrams
- Expand beyond multiply-only microcode
- Add a program-complete or valid-result status output
- Improve observability for debug and bring-up
- Add richer verification results to this document
