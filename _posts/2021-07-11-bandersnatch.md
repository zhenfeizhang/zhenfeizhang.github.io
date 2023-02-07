---
layout: post
title: Introducing the Bandersnatch curve
image:
tags: [snark, curve]
---

The [Bandersnatch curve](https://ethresear.ch/t/introducing-bandersnatch-a-fast-elliptic-curve-built-over-the-bls12-381-scalar-field/9957) 
was discovered by  Simon Masson from [Anoma](https://anoma.network/) and Antonio Sanso from [Ethereum Foundation](https://ethereum.foundation/).
I have [implemented this curve with rust](https://github.com/zhenfeizhang/bandersnatch). The group operation is __40%__ faster than
the Jubjub curve. 
See [here](https://eprint.iacr.org/2021/1152) for a short report on this curve.