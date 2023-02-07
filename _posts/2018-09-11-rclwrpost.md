---
layout: post
title: OnBoard Security Creates Computational Learning With Rounding Over Rings (R-CLWR) Problem
image:
tags: [blog]
---


Originally posted [here](http://blog.onboardsecurity.com/blog/onboard-security-creates-computational-learning-with-rounding-over-rings-r-clwr-problem).

The world is facing an increasing threat from quantum computers.  All widely deployed public key cryptosystems, namely, RSA, ECC and (EC)DH, will be broken due to Shor’s algorithm running on a quantum computer. To mitigate this threat, NIST started a call for proposal to identify cryptographic algorithms that are secure against quantum computers (a.k.a, post-quantum cryptosystems or PQC).

 

NIST Submissions
--------------------

NIST received in total  80 submissions, with 64 submissions still under consideration.  The family of lattice-based solutions are among the most promising solutions. A common approach in lattice-based systems is for the user to take a common reference value “A” and a private value “s”, and make public a value b = As + e that contains s (thanks to the As term) and yet “hides” it (thanks to the e term). If s and e are “large enough”, this effectively hides s, while if s and e are “small enough”, this public value b uniquely identifies s and allows calculation of signatures or shared secrets. Lattice-based cryptography is made possible because there is a wide range of values of s and e that are both large enough in the first sense and small enough in the second sense. Compared to older solutions, such as elliptic curve Diffie-Hellman algorithm, this solution provides quantum resistance. Furthermore, it is provably secure, in the sense that if a and e have particular forms, it can be shown that breaking these cryptosystems (“Ring Learning with Errors cryptosystems”) is as hard as solving an underlying lattice problem on random lattices, i.e. the special form of this problem does not give an attacker any advantage. 

Although this system is provably secure, it is considerably larger than older solutions; a lattice-based key exchange requires around 2000 bytes in each direction, while an elliptic curve-based solution requires less than 100. The goal of many submitted NIST solutions is to minimize this payload size. One possible way is to rely on rounding functions instead of errors. For example, instead of setting b = As +e, one may set b = round(As) which “chops off” the low-order bits and reduces the coefficient sizes.  There are a few Ring Learning with Rounding (R-LWR) based proposals, such as Round2/5, Lizzard and Saber, among the most efficient NIST candidates. 

 

The Problem with Ring Learning with Errors
------------------------------------

However, cryptographic proofs are subtle.  The probabilistic errors become deterministic and this breaks the proof for Ring Learning with Errors cryptosystems. Hence, one will need to make extra assumptions to guarantee the security. This is a major concern for the design of these otherwise efficient systems; the proof for Ring Learning with Errors does not apply to Ring Learning with Rounding (specifically, to “decisional Ring Learning with Rounding”). This problem basically assumes that one cannot distinguish the pairs (A,b) and (A,u) where b = round(As) for some secret s and u = round(U) for some uniformly random element U over the ring. 

Jacob Alperin-Sheriff, a researcher from NIST, has published a  [list](http://crypto-events.di.ens.fr/LATCA/program/alperin-sheriff.pdf)  of open problems in lattice based cryptography with regard to the post quantum cryptography , and this decisional Ring Learning with Rounding problem is one of the most significant open problems he identifies. 




Computational Learning With Rounding over rings (R-CLWR)
------------------------------

Given the importance of this problem, OnBoard Security, in collaboration with researchers from New Jersey Institute of Technology and Chinese Academy of Science, have collaborated to find a solution; Computational Learning With Rounding over rings (R-CLWR) problem. This solution can be viewed as an analogue to the computational Diffie-Hellman problem that bypasses the decisional R-LWR problem. In a nutshell, we are able to show that c = round(Ast) is indistinguishable from round(U), and to further show that although we are not able to build a proof for b = round(As) (i.e., decisional R-LWR), our result on round(Ast) is enough to build a Diffie-Hellman type key exchange scheme. We believe that this result will give great confidence to R-LWR based submissions and will have immediate impact on the NIST deliberations.

There’s one caveat on this: the theorem doesn’t say anything about practical parameter sizes and does not provide a provable security for any proposed parameters in the submissions. However, the same is true of other lattice-based cryptographic systems (such as the well-established Learning with Errors problem. This result is therefore a significant step forward in post quantum cryptography.
