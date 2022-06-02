---
layout: default
title: Highlight of projects
---

# Privacy-preserving machine learning
- ZK for Neural Networks
  - [github](https://github.com/UCSB-TDS/ZEN), [paper](https://eprint.iacr.org/2021/087)
  
# Blockchain x Post-quantum Crypto

- Signature for consensus:
  - Squirrel: Efficient Synchronized Multi-Signatures from Lattices. [github](https://github.com/zhenfeizhang/sync_multi_sig), [paper](https://eprint.iacr.org/2022/694)

- Verifiable random functions for consensus
  - Lattice based construction: [github](https://github.com/zhenfeizhang/lb-vrf), [paper](https://eprint.iacr.org/2020/1222)
  - Hash based construction: [paper](https://eprint.iacr.org/2021/302)

- Signatures for authentication
  - Raptor signature: [github](https://github.com/zhenfeizhang/raptor), [paper](https://eprint.iacr.org/2018/857)
  - Falcon signature: [Falcon-rust](https://github.com/zhenfeizhang/falcon-rust), [Falcon-r1cs](https://github.com/zhenfeizhang/falcon-r1cs).

# Blockchain cryptography
- ZKP architecture and applications
  - Jellyfish: a rust implementation of __PLONK__ zero-knowledge proof system 
    - the most feature-complete and fastest open-source implementation of PLONK;
    - [github](https://github.com/SpectrumXYZ/jellyfish), [press](https://www.espressosys.com/blog/introducing-the-jellyfish-cryptography-library)
    - also comes with a [solidity verifier](https://.github.com/EspressoSystems/cape/tree/main/contracts/contracts/verifier)
  - Manta: Privacy Preserving Decentralized Exchange
    - [website](https://manta.network), [github](https://github.com/Manta-Network), [paper](https://eprint.iacr.org/2020/1607), [press](https://www.theblockcrypto.com/linked/93365/polychain-privacy-dex-manta-seed)
  - Pointproofs: [github](https://github.com/algorand/pointproofs), [paper](https://eprint.iacr.org/2020/419), [press](https://www.algorand.com/resources/blog/pointproofs)

- Signatures
  - BLS signature: [github](https://github.com/algorand/bls_sigs_ref), [crate.io](https://crates.io/crates/bls_sigs_ref), [Internet-Draft](https://tools.ietf.org/html/draft-boneh-bls-signature-00), [press](https://www.algorand.com/resources/blog/first-release-bls-library)
  - Pixel signature: [github](https://github.com/algorand/pixel), [press](https://medium.com/algorand/digital-signatures-for-blockchains-5820e15fbe95)

- Curves
  - Bandersantch curve: [original post](https://ethresear.ch/t/introducing-bandersnatch-a-fast-elliptic-curve-built-over-the-bls12-381-scalar-field/9957), [github](https://github.com/zhenfeizhang/bandersnatch).

# NIST post-quantum cryptography
- 3nd round finalists
  - [Falcon Signature](https://falcon-sign.info/)
  - [NTRU](https://ntru.org)

- 2nd round candidates
  - [Round5 KEM/PKE](https://round5.org/)
  - [LAC](https://eprint.iacr.org/2018/1009.pdf)

- 1st round candidate
  - [pqNTRUSign](https://eprint.iacr.org/2019/1301)
