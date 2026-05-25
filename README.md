# CipherStream-128X
A high-throughput, fully pipelined AES-128 hardware accelerator in Verilog. Packaged as an AXI4-Lite IP core, it ensures deterministic, stall-free encryption and dynamic key management for continuous, secure data streaming in SoC environments. 

# CipherStream-128X: AXI4-Enabled High-Throughput AES-128 Accelerator

## Overview
The CipherStream-128X is a fully unrolled, 10-stage pipelined AES-128 hardware accelerator designed for high-throughput, deterministic encryption in hostile environments. The core is integrated with a custom AXI4-Lite memory-mapped interface, ensuring seamless SoC compatibility and strict protocol adherence. 

## Key Architectural Features

### 1. 10-Stage Unrolled Datapath
The datapath maps the standard AES-128 FIPS-197 algorithm into discrete physical hardware stages. By fully unrolling the pipeline, the system can generate a new 128-bit ciphertext block every valid clock cycle once the pipeline is full. 
* **Combinational S-Boxes:** The SubBytes transformation utilizes distributed combinational logic (LUTs) rather than synchronous BRAM. Instantiating 16 parallel 8-bit S-boxes per round allows the pipeline to cascade SubBytes, ShiftRows, MixColumns, and AddRoundKey into a single, seamless clock cycle.
* **In-Flight Data Tracking:** An 11-bit shift register runs in parallel with the data registers to track valid blocks, safely managing empty "bubbles" in the pipeline.

### 2. Dynamic Key Management & Shadow Registers
The system supports on-the-fly dynamic key updates via the AXI Master without interrupting streaming operations or corrupting data.
* **Shadow Register Architecture:** New keys are written exclusively to a `shadow_key` register, leaving the `active_key` unchanged so in-flight data can finish encrypting safely.
* **Atomic Swap:** When the pipeline empties, the hardware executes a zero-latency swap, copying the shadow key into the active key exactly between data streams.
* **Pipelined Key Expansion:** The active key is fed into 10 parallel key expansion modules to generate the 1408-bit expanded key bus dynamically, pipelined to maximize clock frequency.

### 3. AXI4-Lite Interface & Backpressure Handling
The IP features a custom wrapper to resolve the data width mismatch between the 32-bit AXI4-Lite bus and the 128-bit AES engine.
* **Data Packing/Unpacking:** The interface acts as an accumulation buffer, packing four sequential 32-bit writes into a single 128-bit plaintext block, and unpacking the 128-bit ciphertext output across four 32-bit read operations.
* **Zero-Data-Loss Freezing:** If the downstream processor is busy (AXI backpressure), the wrapper pulls a global `enable` signal low. This instantly freezes the clock enable (CE) pins on all 1,408 flip-flops in the datapath, holding all blocks perfectly in place.

## Performance & Synthesis Results
The design was synthesized targeting the Xilinx Zynq-7000 series architecture (xc7z020clg484-1). 

* **Maximum Operating Frequency (Fmax):** 215.89 MHz
* **Effective System Throughput:** ~6.9 Gbps (6,908.48 Mbps)
* **Total Latency:** 19 clock cycles / 88.0 ns

### Resource Utilization
| Resource Type | Utilized Count | Available | Utilization % |

| **Slice LUTs** | 9,686 | 53,200 | 18.21% |
| **Slice Registers (FFs)** | 3,381 | 106,400 | 3.18% |
| **Block RAM (BRAM)** | 0 | 140 | 0.00% |

*Note: BRAM utilization is intentionally kept at zero. Synthesizing the SubBytes transformations into LUTs guarantees single-cycle combinational evaluation, preventing synchronous read latencies that would bottleneck throughput.*

---
*Developed by Team Ronit (Rohanraj Mallick, Nitin Kumar Rai) for the UDYAM '26 I-CHIP PS-2 Hardware Design Challenge.*
