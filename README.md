## Overview

What if we could default to attack-resistant random generators without excessive
CPU cost? Thanks to hardware acceleration and a new sponge-like generator, this
module generates numbers faster than MT19937 and pcg64_c32 in real-world
benchmarks.

## Related work

AES-CTR (encrypting a counter) is a well-known and easy to implement generator.
It has two known weaknesses:

-   A known-key distinguisher on 10-round, 128-bit AES [https://goo.gl/3xReB9].

-   No forward security/backtracking resistance: compromising the current state
    lets attackers distinguish prior outputs from random.

NIST 800-90a r1 [https://goo.gl/68Fwmv] is a standardized generator that ensures
backtracking resistance, but is not fast enough for a general-purpose generator
(5-10x slower than AES).

## Algorithm

The algorithm has three building blocks:

1)  Reverie [https://eprint.iacr.org/2016/886.pdf] is a sponge-like generator
    that requires a cryptographic permutation. It improves upon "Provably Robust
    Sponge-Based PRNGs and KDFs" by achieving backtracking resistance with only
    a single permutation per buffer.

2)  Simpira v2 [https://eprint.iacr.org/2016/122.pdf] constructs up to 1024-bit
    permutations using an improved Generalized Feistel network with 2-round
    AES-128 functions. This Feistel block shuffle achieves diffusion faster and
    is less vulnerable to sliced-biclique attacks than a Type-2 cyclic shuffle.

3)  "New criterion for diffusion property" [https://goo.gl/mLXH4f] shows that
    the same kind of improved Feistel block shuffle can be extended to 16
    branches, which enables a more efficient 2048-bit permutation.

We combine these by plugging the larger Simpira-like permutation into Reverie.

## Performance

The implementation targets x86 (Westmere), POWER 8 and ARM64. Microbenchmarks
report 1.55 and 2.94 cycles per byte on the first two platforms (faster than
Mersenne Twister). Real-world applications (shuffling and sampling) indicate a
performance advantage over pcg64_c32.

## Security

Randen is empirically random, computationally infeasible to predict and
backtracking-resistant. For more details and benchmarks, please see
the paper "Randen - fast backtracking-resistant random generator with
AES+Feistel+Reverie".

## Usage

`make && bin/randen_benchmark`

Note that the code relies on advanced compiler optimizations. Expect about 1.6x
worse performance than reported when using GCC 4.8, and 3x with Clang 3.4.


This is not an official Google product.
