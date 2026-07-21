# MBSNTT: In-Memory NTT Accelerator

A transistor-level implementation of the MBSNTT 
(Multi-Bit-Serial NTT) processing-in-memory accelerator 
for Number Theoretic Transform (NTT), targeting 
post-quantum cryptographic workloads such as CRYSTALS-Kyber.

Implemented in **Cadence Virtuoso** at **TSMC 28nm**, 
verified via Spectre simulation.

> Based on: Pakala, Chen, and Yang, "MBSNTT: A Highly 
> Parallel Digital In-Memory Bit-Serial Number Theoretic 
> Transform Accelerator," IEEE TVLSI, Feb. 2025.

---

## Background

Lattice-based post-quantum cryptography schemes like 
CRYSTALS-Kyber rely heavily on polynomial multiplication, 
accelerated via the Number Theoretic Transform (NTT). 
Conventional NTT accelerators are bottlenecked by memory 
bandwidth and repeated data movement between processor 
and memory.

MBSNTT addresses this by embedding computation directly 
inside SRAM arrays — the memory is also the compute unit.

---

### Key Components

**Compute Array**
- Custom logic embedded between every two SRAM rows
- Each cycle: selects 1b from B (MSB first via 8:1 Mux)
  and 4b nibble from A segment (via 8:4 Mux)
- 4 AND gates produce 4×1b partial products
- Tree adder (FA+HA) accumulates to 6b partial sum
- Computes all N/2 butterfly B·W multiplications 
  simultaneously in parallel

**NM (Near-Memory) Logic**
- MAC accumulator: shift + 10b add per cycle
- Modulo reduction: two-adder CLA structure,
  keeps running sum < q without dedicated 
  reduction pass
- No Barrett or Montgomery reduction needed

**Modulo Adder**
- Two-adder construction: S = X+Y, S' = X+Y-q
  in parallel; carry-out selects correct result
- Carry-lookahead adder for low area and good 
  throughput

**Constant Geometry NTT Dataflow**
- Read/write addresses identical across all stages
- Outputs stored in NM between stages, avoiding
  memory conflicts
- 512 cycles for interstage routing for any N

---

## Multi-Bit-Serial Multiplication

The core innovation is a divide-and-conquer segment 
multiplication scheme. For a 32×32b multiplication:

- Operands split into four 8b segments
- Segment pairs whose output bits overlap are 
  computed **in parallel** (up to 4 pairs/cycle)
- Within each pair: 8×8b done in 16 cycles 
  (MSB→LSB on B, two cycles per bit: lower 
  nibble then upper nibble of A)
- Horner-style accumulation in NM logic:
  psum ← mod_add(psum, mac) then 
  psum ← mod_add(psum, psum) every odd cycle

**Timing Diagram (32×32b segments):**

| Set | Segment Pairs (parallel)              | Output Bits |
|-----|---------------------------------------|-------------|
| 1   | (A[7:0], B[7:0])                      | 0–15        |
| 2   | (A[15:8],B[7:0]) ∥ (A[7:0],B[15:8])  | 8–23        |
| 3   | (A[23:16],B[7:0]) ∥ ... ∥ (A[7:0],B[23:16]) | 16–31 |
| 4   | All four pairs along main diagonal    | 24–39       |
| 5   | (A[31:24],B[15:8]) ∥ ...             | 32–47       |
| 6   | (A[31:24],B[23:16]) ∥ (A[23:16],B[31:24]) | 40–55  |
| 7   | (A[31:24], B[31:24])                  | 48–63       |

Total: **114 cycles** for complete 32×32b modular 
multiplication at 168 MHz (scalable to 250 MHz 
with tree adder upgrade).

---

## Implementation Details

| Parameter        | Value                        |
|------------------|------------------------------|
| Technology       | TSMC 28nm                    |
| Tool             | Cadence Virtuoso + Spectre   |
| Average Power    | 129.9 pW                     |
| Clock Frequency  | 168 MHz (→ 250 MHz planned)  |
| Clock Cycles     | 114 (32×32b multiplication)  |
| Latency          | 684 ns (→ 456 ns planned)    |
| Modulo Reduction | CLA-based, no Barrett/Montgomery |

### Current Scope
This implementation covers the **B·W modular 
multiplication** unit (compute array + NM logic). 
The modulo adder for A±B·W butterfly completion 
is identified as next step (see Future Work).

---

## Repository Contents

| File | Description |
|------|-------------|
| `TopModule-VLSI.mp4` & `VLSI_Design.mp4` | Video 1 & 2: MBSNTT architecture overview and simulation |
| `MBSNTT.ipynb` | Timing and cycle analysis notebook |
| `HNM.csv` | Hold Noise Margin data |
| `RNM.csv` | Read Noise Margin data |
| `Final_Presentation.pptx` | Project presentation slides |
| `pwr_waveform.png` | Cadence Spectre power simulation output |
| `Final_Poster.png` | Undergraduate Research Showcase poster |

---

## Future Work

- Replace 32-bit ripple carry adder with tree adder
  in NM logic → 250 MHz, 456 ns latency
- Integrate modulo adder to complete full butterfly
  unit (A+B·W and A−B·W)
- Replicate to N/2 parallel units for full NTT pipeline
- Chain log₂N stages with constant geometry routing

---

## Reference

A. Pakala, Z. Chen, and K. Yang, "MBSNTT: A Highly 
Parallel Digital In-Memory Bit-Serial Number Theoretic 
Transform Accelerator," IEEE Transactions on VLSI 
Systems, vol. 33, no. 2, pp. 537–550, Feb. 2025.
doi: 10.1109/TVLSI.2024.3462955
