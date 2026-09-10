
# Direct Memory Access (DMA) Controller – RTL to GDSII ASIC Physical Design Flow

A production-ready, high-throughput Direct Memory Access (DMA) Controller designed in **Verilog HDL** and taken through the complete **ASIC Physical Design Flow (RTL-to-GDSII)** using **Cadence Innovus 21.10** targeting a **28nm CMOS Technology Node** at **500 MHz (2.0 ns clock period)**.

---

## Executive Summary

The primary objective of this project is to implement a hardware-automated DMA controller that offloads data migration tasks from the host processor. By maintaining direct master control over the memory bus, this DMA architecture eliminates CPU cycles wasted on byte-by-byte copies, enabling continuous background memory transfers with single-interrupt completion signaling.

### Architectural Highlights
- **Width Flexibility:** Built-in byte alignment hardware supporting 8-bit, 16-bit, and 32-bit data transfers.
- **Endian Conversion:** Hardware byte-swapping logic supporting seamless Little-Endian $\leftrightarrow$ Big-Endian data conversions on-the-fly.
- **Integrated Staging FIFO:** A $16 \times 32\text{-bit}$ circular synchronous FIFO buffer decoupling read-side and write-side memory access latencies.
- **Error Handling & Protection:** 9-state Finite State Machine with boundary violation detection ($>1023$ words), zero-length transfer traps, and FIFO overflow/underflow recovery.

---

## 1. System Architecture & Block Diagram

The top-level hierarchy (`dma_top`) integrates 8 dedicated functional modules to manage configuration, state sequencing, address calculation, data transformation, buffering, and interrupt management.

```text
                                  +-----------------------+
                                  |  dma_register_file    | (CPU Configuration Interface)
                                  +-----------+-----------+
                                              |
                                  +-----------v-----------+
                                  |        dma_fsm        | (9-State Controller)
                                  +-----+-----------+-----+
                                        |           |
                   +--------------------+           +--------------------+
                   |                                                     |
       +-----------v-----------+                             +-----------v-----------+
       |   source_addr_gen     |                             |     dest_addr_gen     |
       +-----------+-----------+                             +-----------+-----------+
                   |                                                     |
                   +--------------------+           +--------------------+
                                        |           |
                                  +-----v-----------v-----+
                                  |     memory_model      | (1024x32 SRAM)
                                  +-----------+-----------+
                                              |
                                  +-----------v-----------+
                                  |    source_decoder     | (Width / Endianness Alignment)
                                  +-----------+-----------+
                                              |
                                  +-----------v-----------+
                                  |         fifo          | (16x32 Staging Buffer)
                                  +-----------+-----------+
                                              |
                                  +-----------v-----------+
                                  |  interrupt_controller | (IRQ Logic)
                                  +-----------------------+
