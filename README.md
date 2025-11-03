
# 🔁 RDT-Feistel: A Deterministic Recursive-Entropy Mixing System
**Author:** Steven Reid (2025) | **License:** MIT  
**Repository structure:**


RDT-Feistel/
├── rdt_feistel_v1.py
├── rdt_feistel_v3f.py
├── RDT-Feistel.ipynb/
└── README.md

```

---

## 1  Overview
**RDT-Feistel** (Recursive Division Tree Feistel) is a fully deterministic, reversible data-mixing system that produces near-maximal Shannon entropy and full diffusion **without using any randomness**.  
It demonstrates that **recursive geometric structure alone** can reproduce the statistical signatures of cryptographic diffusion.

The system combines:

| Principle | Role |
|------------|------|
| **Recursive Division Tree (RDT)** | Generates a structural permutation of indices based on a recursive depth law \( d(n) \propto (\log n)^{\alpha} \). |
| **Feistel Network** | Provides reversibility and controlled entropy growth across rounds. |
| **ARX Mixing (φ)** | Each round uses additive-rotation-xor operations parameterized by the golden ratio φ ≈ 1.618 to break symmetry. |
| **Recursive Entropy Law** | Models bounded entropy growth \(H_{n+1}=H_n+\alpha(1-2^{-n})\) approaching the 8 bits/byte Shannon limit. |

---

## 2  Scientific Background

RDT-Feistel originated inside the *Recursive Entropy Project* as an experimental test of the **Recursive Entropic Field Theory (REFT)**.  
Its goal was to show that recursive subdivision of an index space could, through deterministic structure alone, evolve an entropy distribution indistinguishable from randomness.

### 2.1  Entropy as Recursive Geometry
For integer n ≥ 2,
\[
\text{depth}(n)=\sum_{k=1}^{m}1,\qquad
x_{k+1}=x_k / \max\!\left(2,\lfloor(\log x_k)^{\alpha}\rfloor\right)
\]
with α ≈ 1.5.  
Sorting indices by `depth(i+2)` builds the RDT permutation P.  
This replaces random tables with a deterministic “geometric” ordering.

### 2.2  Feistel Structure
A classical Feistel round:
```

L, R = R, L XOR F(R)

````
is inherently bijective—perfectly reversible.  
RDT-Feistel embeds its φ-based ARX function F into this framework.

---

## 3  Versions in this Repository

| Version | File | Key characteristics |
|----------|------|--------------------|
| **v1** | `rdt_feistel_v1.py` | Original 2025 formulation: pure RDT permutation + φ-based ARX Feistel. No added diffusion layers. |
| **v3f** | `rdt_feistel_v3f.py` | Final bias-corrected architecture with global cross-byte rotation (histogram flattening) and AES-style MDS diffusion (avalanche restoration). |

Both are 100 % deterministic, reversible, and implemented using only the Python 3 standard library + NumPy (for convenience).

---

## 4  Algorithmic Structure

### 4.1  Round Function F
Each round uses additive–rotation–xor (ARX) logic driven by φ:

\[
F_i = \text{rotl}_8\!\Big((a_i\oplus b_i)+(c_i\oplus d_i),\,r_s\Big)
      \oplus \text{rotl}_8\!\Big((a_i+d_i)\oplus(b_i+c_i),\,r_s'\Big)
\]

Rotation offsets depend on φ · (i + 13 r), ensuring incommensurate angular frequencies that prevent symmetry lock-in.

### 4.2  Recursive Entropy Law
Each Feistel round corresponds to an incremental entropy step:

\[
H_{r+1}=H_r+\alpha(1-2^{-r})
\]
bounded by H ≤ 8 bits / byte.

### 4.3  Enhancements Introduced in v3f
| Enhancement | Purpose | Reversibility |
|--------------|----------|----------------|
| **Global bit-rotation (ROT)** | Rotates the entire bitstream by a fixed offset per round, flattening byte-frequency bias. | Bijective |
| **MDS diffusion layer (MIX)** | Applies an AES-style MixColumns transformation on 16-byte stripes to propagate single-bit changes through all bytes. | Bijective (linear over GF(2⁸)) |

Combined structure:

\[
\text{Round}_r = \text{MIX}\bigl(\text{ROT}\bigl(\text{Feistel}_r(L,R)\bigr)\bigr)
\]

---

## 5  Empirical Results (32 kB structured input)

| Metric | v1 (Original) | v3f (Final) | Target |
|:--------|:---------------:|:-------------:|:------:|
| Entropy (bits/byte) | 7.9936 | **7.9929** | ≈ 8 |
| Avalanche fraction | 0.0000 | **0.4998** | ≈ 0.5 |
| χ² (Uniformity) | 291 | **320.9** | 250–350 |
| Reversibility | ✔ | ✔ | ✔ |
| Randomness source | None | None | — |

**Interpretation:**  
- v1 → Perfect entropy but no diffusion (good structural test).  
- v3f → Perfect diffusion *and* perfect uniformity—statistically indistinguishable from cryptographic diffusion yet fully deterministic.

---

## 6  Potential Applications

| Domain | Use |
|---------|-----|
| **Deterministic seed whitening** | Turn structured inputs into uniform, high-entropy seeds without RNGs. |
| **Reversible compression pre-stage** | Equalizes symbol frequencies before arithmetic or ANS coders while preserving reversibility. |
| **Chaos / entropy simulations** | Models bounded entropy growth laws in discrete data. |
| **Reversible computing** | Provides a deterministic, information-preserving transform usable in energy-efficient pipelines. |
| **Research / teaching** | Demonstrates that statistical randomness can emerge from pure structure. |

---

## 7  Running Benchmarks

### Command line
```bash
python rdt_feistel_v1.py
python rdt_feistel_v3f.py
````

### Sample Output

```
=== RDT-Feistel v3f (global-rot + MDS) ===
Entropy (bits/byte): 7.9929
Avalanche fraction : 0.4998
Chi-square          : 320.9
Time                : 3512.6 ms for 32768 bytes
```

---

## 8  Mathematical Properties

| Property                     | Description                                                           |
| ---------------------------- | --------------------------------------------------------------------- |
| **Bijectivity**              | Feistel + invertible linear maps ⇒ perfect reversibility.             |
| **Determinism**              | No random or external parameters; identical input → identical output. |
| **Information preservation** | One-to-one mapping; ideal for reversible data pipelines.              |
| **Bounded entropy growth**   | Mirrors physical systems where entropy saturates at a finite maximum. |
| **Complexity**               | O(n) runtime per round; trivially parallelizable.                     |

---

## 9  Validation Metrics

For research replication you can test:

* **Shannon Entropy**
  (H = -\sum p_i \log_2 p_i)
* **Chi-square Uniformity**
  (\chi^2 = \sum_i (O_i - E)^2/E)
* **Avalanche Fraction**
  Fraction of bits flipped when one input bit changes.
* **NIST SP 800-22 / Dieharder**
  Optional extended randomness tests (v3f passes baseline suites).

---

## 10  Conceptual Significance

RDT-Feistel shows that:

> *Randomness is not required for entropy.*
> *Geometry alone can generate statistical chaos.*

The transition from v1 → v3f empirically validates the **Recursive Entropy Law**—entropy growth via structural recursion rather than stochastic noise.

---
 License

MIT License © 2025 Steven Reid

Permission is granted to use, copy, modify, and distribute this software for any purpose, provided that the original copyright notice and this permission notice are included in all copies.




