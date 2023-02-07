---
layout: post
title: Closer to Proving the NTRU Assumption
image:
tags: [blog]
---

Originally posted [here](http://blog.onboardsecurity.com/blog/closer-to-proving-the-ntru-assumption).


NTRU is a cryptosystem that uses a special type of polynomial ring. The underlying hardness assumption, known as the NTRU assumption, is that an inverse of a short polynomial (polynomial whose coefficients are very short compared to the modulus q) is indistinguishable from a uniformly random polynomial in this ring. This indistinguishability is crucial in designing a cryptosystem.

For over 20 years, the NTRU assumption remains solid. It has inspired many cryptosystems due to its great efficiency. One of the most significant siblings is known as the Ring-Learning with Error (R-LWE) problem, which spawned several schemes including New Hope key exchange. Generally speaking, R-LWE works on tailored rings. So one can prove the indistinguishability for medium size polynomials (whose coefficients are somewhat short compared to the modulus) and therefore base the hardness of a cryptosystem on a known hard problem, rather than an assumption. This is known as the provable security: if one can break a cryptosystem, one can solve a computationally hard problem (prove by contradiction).

For the sake of completeness, we recall the rings for each setting:

* The NTRU ring: Z_q[x]/(x^N-1) with a modulus q a power of 2 and degree N a prime
* The R-LWE ring: a cyclotomic ring, commonly, Z_q[x]/(x^N+1) with N a power of 2 and q a prime that congruents to 1 modulo 2N

The paper "Pseudorandomness of Ring-LWE for Any Ring and Modulus" will appear at STOC 2017, one of a handful most prestige conferences in computer science. In the paper, Chris Peikert, Oded Regev and Noah Stephens-Davidowitz proved that R-LWE is indeed provable secure for any rings, not limited to the R-LWE ring. 

As a typical example, it showed that R-LWE over the prime cyclotomic field Z_q[x]/(x^(N-1) + x^(N-2) + ... + x + 1) is hard.
Note that NTRU is defined over (x^N-1) which is essentially (x^(N-1) + x^(N-2) + ... + x + 1)*(x-1) for a proper choice of N.
It strongly suggests that the choice of Z_q[x]/(x^N-1) is as good as any other ring. The NTRU ring is not a weakness to the systems, rather, a means to achieve better performance in implementation.

Provable security indicates that an attacker must solve the underlying hard problem in order to break the cryptosystem. It doesn't take into account other attacks, such as side-channel attacks. NTRU and its associated signing algorithm, pqNTRUsign, have undergone years of scrutiny and the NTRU Challenge without being broken by any type of attack. Proving the NTRU assumption would just add to NTRU's solid reputation.
