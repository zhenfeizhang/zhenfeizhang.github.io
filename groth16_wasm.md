
# Faster Client-Side Groth16 Proving in the Browser

_approx. 4 min to read_

Zero-knowledge proofs are increasingly used in blockchain and privacy-preserving applications, but generating Groth16 proofs client-side in the browser has traditionally been slow and impractical for real-world use. Previous approaches compiled Go ([gnark](https://github.com/Consensys/gnark)) or Rust ([arkworks](https://github.com/arkworks-rs)) Groth16 provers directly to WASM, but these run in a single thread, resulting in significant performance penalties. For example, proving a typical Zcash-like circuit (~2^16 constraints) could take about 1 minute in the browser—far too slow for good user experience or timely blockchain operations, where block finality is often just 5 seconds.

## The Problem: Single-Threaded WASM Bottleneck
Go/WASM and Rust/WASM environments do not support multi-threading out of the box. This means that all heavy cryptographic operations—especially FFTs and MSMs required for Groth16—run serially, making client-side proving prohibitively slow.

## The Solution: Parallelized Groth16 Prover via C++/Emscripten
To overcome this, I split the proving process:

## Witness Extraction (Go/gnark):
Use Go/gnark only to extract the witness and intermediate data (gnark_output). Go is well-suited for circuit logic and witness generation, but not for parallel computation in WASM.

## Parallel Proof Generation (C++/Emscripten):
Compile the [MCL library](https://github.com/herumi/mcl) (for BN254 pairing operations) using Emscripten, which supports WASM threading. Write glue code in C++ to call MCL and implement the FFT and MSM steps required for Groth16. The main API is:

```c
void groth16_e2e(const uint8_t *gnark_witness, uint8_t * serialized_groth16_proof);
```

This function takes the witness blob and outputs the proof blob, leveraging multi-threading for speed.

Results: 12x Speedup, Proofs in 5 Seconds
With this architecture, I achieved a 12x speedup over single-threaded WASM approaches. Now, a browser can generate a Groth16 proof for a large circuit in under 5 seconds—fast enough for real-time blockchain operations and a much better user experience.

## How It Works
Step 1: Use Go/WASM APIs to generate the witness and extract intermediate data.
Step 2: Pass the witness to the C++/Emscripten WASM module.
Step 3: The module runs groth16_e2e, processes the witness in parallel, and returns the proof.
This approach decouples circuit logic from heavy cryptographic computation, allowing each part to run in its optimal environment.

