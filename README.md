
# Cryptography by Neural Network: Neural-Probabilistic Obfuscation Protocol (NPOP)

A proof-of-concept exploring data obfuscation through deterministic pseudo-random weight generation in feedforward neural networks combined with HMAC-SHA256 seed derivation.

## Overview

This project demonstrates probabilistic text encoding using 256-dimensional vector representations. For each byte of input data, a feedforward neural network (256 input units, 32 hidden units, 256 output units) is instantiated deterministically using a seed derived via HMAC-SHA256.

Rather than transmitting bytes directly or through conventional ciphertexts, the data is represented as a Softmax probability distribution across all 256 possible byte indices.

---

## System Architecture

```text
 [ Input Byte ] + [ Shared Secret + Nonce ]
        │                       │
        │                  HMAC-SHA256
        │                       │
        │               [ PRNG Seed ]
        │                       │
        │           He/Kaiming Initialization
        │            W1 (256x32), W2 (32x256)
        │                       │
        └──────────► [ Neural Forward Pass ]
                                │
                      ReLU & Scaled Softmax
                                │
                      [ Vector p in R^256 ]
```

---

## Mathematical Formulation

### 1. Cryptographic Seed Derivation

For byte position $i$, a deterministic 256-bit seed is generated via HMAC-SHA256:

$$
\text{Seed}_i = \text{HMAC-SHA256}(K_{\text{secret}}, \text{Scheme} \parallel \text{Nonce} \parallel i)
$$

This seed initializes a pseudo-random number generator (NumPy `default_rng`), guaranteeing identical weight matrix generation on both sender and receiver sides.

### 2. Forward Pass and Probability Generation

Weight matrices are initialized using He (Kaiming) normal initialization suited for ReLU activations:

$$
W_1 \sim \mathcal{N}\left(0, \sqrt{\frac{2}{256}}\right), \quad W_2 \sim \mathcal{N}\left(0, \sqrt{\frac{2}{32}}\right)
$$

The forward pass computes hidden activations with a ReLU nonlinearity, followed by a scaled Softmax layer:

$$
h = \max(W_1 + b_1, 0)
$$

$$
z = (h \cdot W_2 + b_2) \times \gamma \quad (\text{where } \gamma = 60.0)
$$

$$
p_j = \frac{e^{z_j - \max(z)}}{\sum_{k=0}^{255} e^{z_k - \max(z)}}
$$

### 3. Decoding Strategy

The receiver reconstructs candidate codebooks for all 256 potential byte values at position $i$ using the shared key and nonce, recovering the original byte $\hat{b}$ by Chebyshev distance ($L_\infty$ norm) minimization:

$$
\hat{b} = \arg\min_{b \in [0, 255]} \Vert \text{CB}(b) - v_{\text{received}} \Vert_\infty
$$

---

## Theoretical Analysis and Limitations

| Metric / Aspect | Observation |
| :--- | :--- |
| **Data Expansion Ratio** | Transmitting 256 floating-point values per byte results in a payload size of $\approx 2.5 \text{ KB}$ per character (expansion factor $> 2500:1$). |
| **Model Role** | The network performs no statistical learning or inference in the traditional machine learning sense; it functions as a non-linear Pseudo-Random Function (PRF). |
| **Floating-Point Portability** | Due to potential IEEE 754 floating-point discrepancies across hardware targets, probability vectors are rounded to 10 decimal places (`np.round`). |
| **Security Model** | Confidentiality relies entirely on HMAC-SHA256 and PRNG state secrecy, not on neural network properties. |

---

## Requirements and Usage

### Dependencies

* Python 3.10+
* `numpy`
* `tkinter` (standard library GUI module)

```bash
pip install numpy
```

### Execution

Run the main script to launch the interactive demonstration GUI:

```bash
python main.py
```

---

## GUI Features

* **Transmission Tab**: Interactive text input, shared key generation, packet transmission, and real-time decoding latency benchmarks.
* **Intercepted Vectors Tab**: Inspection of raw transmitted JSON packets and probabilistic vector representations.
* **Weights & Biases Tab**: Direct inspection of generated internal matrices ($W_1, b_1, W_2, b_2$) and position fingerprints.

---

## Future Research Directions

* **Vectorized Decoding**: Parallelizing codebook evaluation to decrease decoding overhead for larger payloads.
* **Fixed-Point Arithmetic**: Replacing 64-bit floating point matrix operations with 32-bit integer quantization to guarantee cross-platform bit-exactness.
* **Dimensionality Reduction**: Evaluating top-$k$ sparse probability transmission instead of full 256-element arrays.

---

## License

Distributed under the MIT License.

## Warning
This files is a simulation and he can use for encrypt really communications
