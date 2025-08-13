# Fast Client-Side Groth16 Prover: Zcash Circuit in Under 3 Seconds

__tl;dr__: This framework enables proving 2^16 R1CS constraints in under 3 seconds in the browser, fully compatible with Gnark.

## Background and Motivation
Zero-knowledge proofs are increasingly important in blockchain and privacy-preserving applications. However, generating Groth16 proofs client-side in the browser has been slow and impractical. Previous approaches compiled Go ([gnark](https://github.com/Consensys/gnark)) or Rust ([arkworks](https://github.com/arkworks-rs)) Groth16 provers to WASM, but these run single-threaded, causing significant performance penalties. For example, proving a typical [Zcash-like circuit](https://zips.z.cash/protocol/protocol.pdf) (~2^16 constraints) could take about 1 minute in the browser—far too slow for a good user experience or timely blockchain operations, where block finality is often just 10 seconds.



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

<div style="display:flex; justify-content:center; margin:2rem 0;">
<!-- <svg width="650" height="260" viewBox="0 0 650 260" xmlns="http://www.w3.org/2000/svg">
    <rect x="40" y="60" width="220" height="120" rx="16" fill="#dbeafe" stroke="#2563eb" stroke-width="2"/>
    <text x="150" y="85" text-anchor="middle" font-size="18" font-weight="bold" fill="#2563eb">Go (gnark)</text>
    <text x="150" y="110" text-anchor="middle" font-size="14" fill="#1e293b">Step 1: Circuit Logic</text>
    <text x="150" y="130" text-anchor="middle" font-size="14" fill="#1e293b">Step 2: Witness Generation</text>
    <text x="150" y="150" text-anchor="middle" font-size="14" fill="#1e293b">Witness Extraction</text>
    <text x="150" y="170" text-anchor="middle" font-size="13" fill="#64748b">(Go handles steps 1 & 2)</text>
    <rect x="390" y="60" width="220" height="120" rx="16" fill="#dbeafe" stroke="#2563eb" stroke-width="2"/>
    <text x="500" y="85" text-anchor="middle" font-size="18" font-weight="bold" fill="#2563eb">C (WASM)</text>
    <text x="500" y="110" text-anchor="middle" font-size="14" fill="#1e293b">Step 3: FFT, MSM</text>
    <text x="500" y="130" text-anchor="middle" font-size="14" fill="#1e293b">Step 4: Proof Computation</text>
    <text x="500" y="150" text-anchor="middle" font-size="14" fill="#1e293b">Step 5+: Multi-threading</text>
    <text x="500" y="170" text-anchor="middle" font-size="13" fill="#64748b">(C handles steps 3 to end)</text>
    <polygon points="260,120 390,120 380,110 380,130" fill="#2563eb"/>
    <text x="325" y="105" text-anchor="middle" font-size="13" fill="#2563eb">Serialized Data</text>
    <text x="325" y="125" text-anchor="middle" font-size="13" fill="#2563eb">(witness/intermediate)</text>
    <text x="325" y="40" text-anchor="middle" font-size="16" font-weight="bold" fill="#1e293b">Task Split Between Go and C</text>
</svg> -->
    <svg width="900" height="260" viewBox="0 0 900 260" xmlns="http://www.w3.org/2000/svg">
        <!-- Go Block -->
        <rect x="40" y="60" width="220" height="120" rx="16" fill="#dbeafe" stroke="#2563eb" stroke-width="2"/>
        <text x="150" y="85" text-anchor="middle" font-size="18" font-weight="bold" fill="#2563eb">Go (gnark)</text>
        <text x="150" y="110" text-anchor="middle" font-size="14" fill="#1e293b">Step 1: Circuit Logic</text>
        <text x="150" y="130" text-anchor="middle" font-size="14" fill="#1e293b">Step 2: Witness Generation</text>
        <text x="150" y="150" text-anchor="middle" font-size="14" fill="#1e293b">Witness Extraction</text>
        <text x="150" y="170" text-anchor="middle" font-size="13" fill="#64748b">(Go handles steps 1 & 2)</text>
        <!-- C Block -->
        <rect x="640" y="60" width="220" height="120" rx="16" fill="#dbeafe" stroke="#2563eb" stroke-width="2"/>
        <text x="750" y="85" text-anchor="middle" font-size="18" font-weight="bold" fill="#2563eb">C (WASM)</text>
        <text x="750" y="110" text-anchor="middle" font-size="14" fill="#1e293b">Step 3: FFT, MSM</text>
        <text x="750" y="130" text-anchor="middle" font-size="14" fill="#1e293b">Step 4: Proof Computation</text>
        <text x="750" y="150" text-anchor="middle" font-size="14" fill="#1e293b">Step 5+: Multi-threading</text>
        <text x="750" y="170" text-anchor="middle" font-size="13" fill="#64748b">(C handles steps 3 to end)</text>
        <!-- Arrow from Go to C -->
        <defs>
            <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto">
                <polygon points="0 0, 10 3.5, 0 7" fill="#2563eb" />
            </marker>
        </defs>
        <line x1="260" y1="120" x2="640" y2="120" stroke="#2563eb" stroke-width="4" marker-end="url(#arrowhead)" />
        <text x="450" y="90" text-anchor="middle" font-size="13" fill="#2563eb">Serialized Data</text>
        <text x="450" y="108" text-anchor="middle" font-size="13" fill="#2563eb">(witness/intermediate)</text>
        <text x="450" y="40" text-anchor="middle" font-size="16" font-weight="bold" fill="#1e293b">Task Split Between Go and C</text>
    </svg>
</div>

Go/gnark is used only to extract the witness and intermediate data (`gnark_output`). Go is well-suited for circuit logic and witness generation.
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

### End-to-End Proving Time


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

## Future Directions

---

## Next Steps

- Since parallelization is a low-hanging fruit, the next step is to use WebGPU for further acceleration.
- These techniques can also be applied to Plonkish and GKR-type proving systems.