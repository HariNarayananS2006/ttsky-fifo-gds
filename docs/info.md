<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

An 8-bit wide, 16-entry synchronous FIFO with full and empty flags. The layout was hardened by hand with OpenLane 2 into a 1x1 tile and submitted as a custom GDS.

- Write: when `WR_EN` (uio[0]) is high on a rising clock edge and the FIFO is not full, the byte on `ui[7:0]` is stored.
- Read: when `RD_EN` (uio[1]) is high on a rising clock edge and the FIFO is not empty, the oldest byte appears on `uo[7:0]` just after that edge. `uo[7:0]` holds its value until the next read.
- `FULL` (uio[2]) and `EMPTY` (uio[3]) come straight from flip-flops and update on every clock edge.
- A write when full, or a read when empty, is ignored. If the FIFO is empty and both enables are high, only the write happens. If it is full and both are high, only the read happens.
- `rst_n` goes through a 2-flop synchronizer. Hold it low for at least 3 clock cycles. The FIFO is empty 2 cycles after reset is released.

Inside, it uses one-hot read and write pointers and a thermometer-coded occupancy register, so there is no counter or comparator on the critical path. The storage is 128 plain flip-flops.

## How to test

Step the clock manually from the demo board's RP2350, so that each write or read happens exactly once. Holding `WR_EN` high for N clock cycles writes N times.

1. Hold `rst_n` low for a few clocks, then release it and give 2 more clocks. Check that `EMPTY` = 1 and `FULL` = 0.
2. Write: set `ui[7:0]` to a byte, set `WR_EN` = 1, give one clock pulse, then set `WR_EN` = 0. Repeat for more bytes. After 16 writes, `FULL` = 1.
3. Read: set `RD_EN` = 1, give one clock pulse, then set `RD_EN` = 0, and read the byte on `uo[7:0]`. Bytes come out in the order they were written. After the last read, `EMPTY` = 1.

## External hardware

None. The demo board's RP2350 can drive the inputs and read the outputs.