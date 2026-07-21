# In-Memory_NTT_Accelerator
This repository documents the transistor-level implementation of MBSNTT, a digital processing-in-memory NTT accelerator for post-quantum cryptography. The design embeds multi-bit-serial modular multiplication logic directly between SRAM rows in Cadence Virtuoso, computing all N/2 butterfly units in parallel with no Barrett or Montgomery reduction.
