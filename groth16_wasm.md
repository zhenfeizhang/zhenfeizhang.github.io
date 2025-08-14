# Fast Client-Side Groth16 Prover: Zcash Circuit in Under 3 Seconds

__tl;dr__: This framework enables proving 2^16 R1CS constraints in under 3 seconds in the browser, fully compatible with Gnark.

## Background and Motivation
Zero-knowledge proofs (ZKPs) are increasingly vital for blockchain and privacy-preserving applications. Yet, generating Groth16 proofs directly in the browser has historically been slow and impractical. Previous efforts to compile Go ([gnark](https://github.com/Consensys/gnark)) or Rust ([arkworks](https://github.com/arkworks-rs)) Groth16 provers to WASM resulted in single-threaded execution, leading to significant performance bottlenecks. For instance, proving a typical [Zcash-like circuit](https://zips.z.cash/protocol/protocol.pdf) with ~2^16 constraints could take about one minute in the browser—far too slow for a good user experience or for timely blockchain operations, where block finality is often just 10 seconds. As a result, private payment protocols on Ethereum are forced to rely on relayers (compromising privacy) or use fixed-size notes to work around these limitations.


## Parallelization for Browser Performance
The first step for browser acceleration is parallelization. Inspired by [this article](https://web.dev/articles/webassembly-threads), I found the Rust approach complex, with thread pools managed via the WASM caller and multiple rounds of JS-WASM interaction. The C++ method is more natural: we can write parallelized code in C and wrap it with a simple API, requiring only a single call from WASM.

The next step is to extract the computation-heavy components from Gnark and offload them to C. Below is a pseudocode for the Groth16 proving process:
```rust
fn groth16_prove(circuit: &Circuit, witness: &[Fr], proving_key: &ProvingKey) -> Proof {
    // Step 1: Setup phase outputs (assumed already completed)
    // proving_key contains: α, β, δ, {βu_i(x) + αv_i(x) + w_i(x)}/δ for i ∈ I_mid
    // and evaluation points for polynomials u_i(x), v_i(x), w_i(x)
    
    // Step 2: Witness extension
    let extended_witness = extend_witness(circuit, witness);
    // extended_witness = (w_0, w_1, ..., w_m) where w_0 = 1
    
    // Step 3: Polynomial evaluation using MSM
    let u_eval = msm(&proving_key.u_query, &extended_witness);
    let v_eval = msm(&proving_key.v_query, &extended_witness);
    let w_eval = msm(&proving_key.w_query, &extended_witness);
    
    // Step 4: Generate random values
    let r = Fr::rand(&mut rng);
    let s = Fr::rand(&mut rng);
    
    // Step 5: Compute proof elements
    // A = α + u_eval + r*δ
    let a = proving_key.g1_alpha + (u_eval * G1::generator()) + (r * proving_key.g1_delta);
    
    // B = β + v_eval + s*δ
    let b_g1 = proving_key.g1_beta + (v_eval * G1::generator()) + (s * proving_key.g1_delta);
    let b_g2 = proving_key.g2_beta + (v_eval * G2::generator()) + (s * proving_key.g2_delta);
    
    // Step 6: Compute quotient polynomial H(x) using FFT
    let h_coeffs = compute_h(&extended_witness, circuit);
    
    // Evaluate H at secret point using MSM
    let h_eval = msm(&proving_key.h_query, &h_coeffs);
    
    // Compute middle term using MSM
    let mid_term = msm(&proving_key.l_query, &extended_witness[i_mid]);
    
    let c = h_eval + (s * a) + (r * b_g1) - (r * s * proving_key.g1_delta) + mid_term;

    return Proof{ a, b_g2, c}
}
```
<svg width="900" height="400" xmlns="http://www.w3.org/2000/svg">
    <text x="450" y="20" font-size="22" text-anchor="middle" fill="#333" font-weight="bold">Groth16 In browser Prover Architecture</text>
  <defs>
    <filter id="shadow" x="-20%" y="-20%" width="140%" height="140%">
      <feDropShadow dx="0" dy="2" stdDeviation="2" flood-color="#888" flood-opacity="0.3"/>
    </filter>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#333"/>
    </marker>
  </defs>
  <!-- Layer 1: App -->
  <rect x="375" y="30" width="150" height="50" rx="20" fill="#fffbe6" stroke="#fbc02d" stroke-width="2" filter="url(#shadow)"/>
  <text x="450" y="60" font-size="20" text-anchor="middle" fill="#b8860b" font-weight="bold">App</text>  
  <!-- Layer 2: Browser -->
  <rect x="375" y="130" width="150" height="50" rx="20" fill="#e3f2fd" stroke="#1976d2" stroke-width="2" filter="url(#shadow)"/>
  <text x="450" y="160" font-size="20" text-anchor="middle" fill="#1976d2" font-weight="bold">Browser</text>  
  <!-- Layer 3: gnark and C/Wasm -->
  <rect x="150" y="270" width="150" height="50" rx="20" fill="#e8f5e9" stroke="#388e3c" stroke-width="2" filter="url(#shadow)"/>
  <text x="225" y="300" font-size="18" text-anchor="middle" fill="#388e3c" font-weight="bold">gnark</text>  
  <rect x="600" y="270" width="150" height="50" rx="20" fill="#f3e5f5" stroke="#7b1fa2" stroke-width="2" filter="url(#shadow)"/>
  <text x="675" y="300" font-size="18" text-anchor="middle" fill="#7b1fa2" font-weight="bold">C/Wasm</text>  
  <!-- Arrows -->
  <!-- App -> Browser: circuit input -->
  <path d="M450,80 C450,110 450,110 450,130" stroke="#1976d2" stroke-width="3" fill="none" marker-end="url(#arrowhead)"/>
  <text x="455" y="115" font-size="14" fill="#1976d2">1. circuit input</text>  
  <!-- Browser -> gnark: witness (curved left, downward) -->
    <!-- Browser -> gnark: witness (curved right, downward) -->
    <path d="M400,180 C420,220 300,250 225,270" stroke="#388e3c" stroke-width="3" fill="none" marker-end="url(#arrowhead)"/>
    <text x="320" y="245" font-size="14" fill="#388e3c" text-anchor="middle">2. witness</text>
    <!-- gnark -> Browser: circuit description (curved left, upward, wider) -->
    <path d="M225,270 C180,220 320,200 400,180" stroke="#1976d2" stroke-width="3" fill="none" marker-end="url(#arrowhead)"/>
    <text x="270" y="200" font-size="14" fill="#1976d2" text-anchor="middle">1. circuit description</text>
  <!-- gnark -> C/Wasm: gnark_output (straight) -->
  <line x1="300" y1="295" x2="600" y2="295" stroke="#7b1fa2" stroke-width="3" marker-end="url(#arrowhead)"/>
  <text x="450" y="285" font-size="14" fill="#7b1fa2" text-anchor="middle">3. gnark output</text>  
  <!-- C/Wasm -> Browser: proof (curved up) -->
  <path d="M675,270 C675,220 600,180 525,180" stroke="#1976d2" stroke-width="3" fill="none" marker-end="url(#arrowhead)"/>
  <text x="630" y="210" font-size="14" fill="#1976d2" text-anchor="middle">4. Groth16 proof</text>
</svg>


Go/gnark is used only to extract the witness and intermediate data (`gnark_output`). Go is well-suited for circuit logic and witness generation.
Importantly, this approach allows us to reuse all existing gnark circuits, ensuring compatibility and flexibility for a wide range of applications.
To be precise, we modify Gnark's proving function so that when an FFT (inside `compute_h`) or MSM call is made, the input to these calls is serialized into a buffer and the rest of the computation is skipped. This modified process is called WitnessExtraction. The entry point in Go remains almost identical:
```go
// Prove generates the proof of knowledge of a r1cs with full witness (secret + public part).
func Prove(
    r1cs *cs.R1CS, 
    pk *ProvingKey, 
    fullWitness witness.Witness, 
    opts ...backend.ProverOption) (*Proof, error) 

// ExtractIntermediateData extracts intermediate data from the proving process.
func ExtractIntermediateData(
    r1cs *cs.R1CS, pk *ProvingKey, 
    fullWitness witness.Witness, 
    opts ...backend.ProverOption) (*GnarkOutput, error)
```

Next, we pass this buffer to C and utilize C's pthreads, which can be mapped to WASM threads via EMCC. The `groth16_prove` function exposes the following API:

```c
void groth16_proof(const uint8_t *gnark_witness, uint8_t * serialized_groth16_proof);
```

This function takes the witness blob and outputs the proof blob, leveraging multi-threading for speed.
We then compile this function into WASM and achieve natural multi-thread speedup from pthreads.

## Performance Benchmarks
---
<div style="overflow-x:auto; margin: 1.5rem 0;">
<table style="min-width:600px; width:100%; text-align:center;">
        <caption style="caption-side:top; font-weight:bold; padding:0.5rem;">
            Micro benchmark for Multi-scalar-multiplications<br>
            <span style="font-weight:normal; color:#64748b; font-size:0.98em;">in browser, over bn254 G1</span>
        </caption>
    <thead>
        <tr>
            <th>MSM size</th>
            <th>Single-threaded (ms)</th>
            <th>Multi-threaded (ms)</th>
            <th>Speedup</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>128</td><td>30.60</td><td>5.34</td><td>5.73</td></tr>
        <tr><td>256</td><td>47.29</td><td>7.76</td><td>6.09</td></tr>
        <tr><td>512</td><td>90.39</td><td>12.46</td><td>7.26</td></tr>
        <tr><td>1024</td><td>176.70</td><td>18.89</td><td>9.36</td></tr>
        <tr><td>2048</td><td>351.36</td><td>29.33</td><td>11.99</td></tr>
        <tr><td>4096</td><td>702.96</td><td>61.59</td><td>11.41</td></tr>
        <tr><td>8192</td><td>1398.75</td><td>96.91</td><td>14.43</td></tr>
        <tr><td>16384</td><td>2841.03</td><td>187.88</td><td>15.12</td></tr>
        <tr><td>32768</td><td>5602.84</td><td>369.93</td><td>15.15</td></tr>
        <tr><td>65536</td><td>11178.83</td><td>740.21</td><td>15.10</td></tr>
    </tbody>
</table>
</div>

Prior to optimization, single-threaded performance was a major bottleneck: a single G1 MSM took 11 seconds, and Groth16 requires 4 G1 MSMs and 1 G2 MSM, resulting in over 60 seconds for proof generation. By leveraging multithreading across 16 cores, our optimized implementation achieves nearly a 16x speedup, dramatically reducing total proving time and making client-side Groth16 practical in the browser.

---
<div style="overflow-x:auto; margin: 1.5rem 0;">
<table style="min-width:400px; width:100%; text-align:center;">
    <caption style="caption-side:top; font-weight:bold; padding:0.5rem;">Groth16 E2E In browser time for 2^16 constraints</caption>
    <thead>
        <tr>
            <th>Step</th>
            <th>Time (ms)</th>
            <th>Language</th>
        </tr>
    </thead>
    <tbody>
    <tr><td>Witness Extraction</td><td>649.57</td><td>Go</td></tr>
    <tr><td>Deserialize</td><td>172.64</td><td>C++</td></tr>
    <tr><td>Compute H</td><td>635.30</td><td>C++</td></tr>
    <tr><td>Compute Bs1</td><td>238.82</td><td>C++</td></tr>
    <tr><td>Compute Ar</td><td>205.59</td><td>C++</td></tr>
    <tr><td>Compute Krs</td><td>475.94</td><td>C++</td></tr>
    <tr><td>Compute Bs2</td><td>475.18</td><td>C++</td></tr>
    <tr><td><b>Total (Go + C++)</b></td><td><b>2865.09</b></td><td>Go + C++</td></tr>
    </tbody>
</table>
</div>

For end-to-end timing, note that computing BS1, Ar, and related steps takes less time than the corresponding MSMs of the same size in the previous benchmark. This is primarily because the witness is sparse—most scalars are less than 254 bits—making the underlying scalar multiplications significantly faster in practice.

## Future Directions

---

## Next Steps

With parallelization already providing substantial speedups, the next frontier is leveraging WebGPU for even greater acceleration and scalability. Looking ahead, these optimization strategies are also highly relevant for other proof systems, such as Plonkish and GKR, enabling fast client-side proving across a broader range of protocols.