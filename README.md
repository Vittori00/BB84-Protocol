# BB84 Protocol – Quantum Key Distribution

A simulation and analysis of the **BB84 Quantum Key Distribution (QKD)** protocol, implemented with IBM's [Qiskit](https://qiskit.org/) framework. The project studies the robustness of the protocol under different key lengths, quantum channel noise conditions, and eavesdropping scenarios.

---

## Table of Contents

1. [Overview](#overview)
2. [What is the BB84 Protocol?](#what-is-the-bb84-protocol)
3. [Project Features](#project-features)
4. [Repository Structure](#repository-structure)
5. [Technology Stack](#technology-stack)
6. [Getting Started](#getting-started)
7. [Implementation Details](#implementation-details)
8. [Analysis Scenarios](#analysis-scenarios)
9. [Key Metrics](#key-metrics)
10. [Visualizations](#visualizations)
11. [References](#references)

---

## Overview

This project provides a full quantum-circuit implementation of the BB84 protocol and a statistical performance analysis across a wide range of conditions. By running the Jupyter notebook `BB84_Vittori_603188.ipynb`, you can:

- Simulate the quantum key exchange between **Alice** (sender) and **Bob** (receiver).
- Introduce a third party, **Eve** (eavesdropper), and measure how her presence affects key quality.
- Apply three types of **quantum channel noise** and observe their impact on the mismatch rate.
- Analyze how **key length** affects protocol reliability and Eve-detection probability.

---

## What is the BB84 Protocol?

BB84 (Bennett & Brassard, 1984) is the first and one of the most well-known **quantum key distribution** protocols. It allows two parties to generate a shared secret key whose security is guaranteed by the laws of quantum mechanics.

### Core Steps

1. **Alice's preparation** – Alice randomly chooses a bit value (0 or 1) and a basis (rectilinear `+` or diagonal `×`), then encodes the bit into a qubit using that basis.
2. **Quantum transmission** – The qubit is sent through a quantum channel to Bob.
3. **Bob's measurement** – Bob randomly chooses a basis and measures the incoming qubit.
4. **Sifting** – Alice and Bob publicly compare their bases (not their bits). They discard bits where they used different bases; the remaining bits form the **raw sifted key**.
5. **Error detection** – A subset of sifted bits is compared. A high error rate signals the presence of a noisy channel or an eavesdropper.

### Why is it Secure?

Any attempt by Eve to intercept the qubit forces her to measure it, inevitably collapsing its quantum state. When she guesses the wrong basis (~50% of the time) and re-sends the qubit, she introduces detectable errors in Alice and Bob's sifted key. By checking a sample of their final key, Alice and Bob can estimate the probability that Eve went undetected.

---

## Project Features

- ✅ **Full BB84 circuit** built with Qiskit (2 qubits without Eve, 3 qubits with Eve)
- ✅ **Eavesdropper simulation** – Eve intercepts, measures, and re-prepares qubits
- ✅ **Three quantum noise models**: bit-flip (X), phase-flip (Z), bit-phase-flip (Y)
- ✅ **Six key-length scenarios**: 50, 100, 300, 700, 1000, 1500 initial bits
- ✅ **Statistical analysis** – repeated simulations with 95% confidence intervals
- ✅ **Eve detection probability** – combinatorial calculation of `P_undetected(k)` as a function of the fraction of bits checked
- ✅ **Rich visualizations** – mismatch/keep-rate curves and detection probability plots exported as high-resolution PNGs

---

## Repository Structure

```
BB84-Protocol/
├── BB84_Vittori_603188.ipynb          # Main Jupyter notebook – simulation & analysis
├── BB84-Quantum-Key-Distribution.pdf  # Reference paper on the BB84 protocol
└── README.md                          # This file
```

---

## Technology Stack

| Tool / Library | Role |
|---|---|
| **Python 3** | Primary programming language |
| **Qiskit** | Quantum circuit construction and execution |
| **qiskit-aer** | High-performance quantum simulator with noise models |
| **NumPy** | Numerical computations |
| **Matplotlib** | Plotting and visualization |
| **SciPy** | Combinatorial statistics (`scipy.special.comb`) |

Dependencies are installed directly inside the notebook:

```python
!pip install qiskit qiskit-aer matplotlib numpy pylatexenc
```

---

## Getting Started

### Prerequisites

- Python 3.8 or higher
- Jupyter Notebook or JupyterLab

### Run the Notebook

```bash
# Clone the repository
git clone https://github.com/Vittori00/BB84-Protocol.git
cd BB84-Protocol

# Install dependencies
pip install qiskit qiskit-aer matplotlib numpy scipy pylatexenc jupyter

# Launch Jupyter
jupyter notebook BB84_Vittori_603188.ipynb
```

Execute the cells from top to bottom. Each section is self-contained and produces its own plots and printed results.

---

## Implementation Details

### Quantum Circuit

The BB84 circuit is built with the function `build_bb84(with_eve=False, draw=False)` and uses the following registers:

| Register | Description |
|---|---|
| `crA` | Alice's random bit (0 or 1) |
| `crAb` | Alice's basis choice (0 = rectilinear, 1 = diagonal) |
| `crB` | Bob's basis choice |
| `crM` | Bob's measurement result |
| `crE` | Eve's basis choice *(only when `with_eve=True`)* |
| `crME` | Eve's measurement result *(only when `with_eve=True`)* |

Random bits are generated on the quantum computer itself using a Hadamard gate followed by a measurement, guaranteeing true 50/50 randomness.

### Basis Encoding

- **Rectilinear basis** (`+`): qubit is in state `|0⟩` (bit = 0) or `|1⟩` (bit = 1).
- **Diagonal basis** (`×`): qubit is in state `|+⟩` (bit = 0) or `|−⟩` (bit = 1), obtained by applying a Hadamard gate after the rectilinear encoding.

### Sifting Logic

After simulation, the `simulate()` function processes the classical register results to:
1. Retain only bits where Alice and Bob chose the **same basis** (~50% of shots).
2. Compute the **mismatch ratio** – the fraction of kept bits where Alice's bit ≠ Bob's bit.

### Eve's Interception

When `with_eve=True`, a third qubit is added. Eve intercepts the qubit mid-circuit:
1. Randomly chooses a basis.
2. Measures the qubit (collapsing its state).
3. Re-prepares a new qubit based on her measurement and sends it to Bob.

When Eve guesses the wrong basis, the qubit she forwards is incorrect ~50% of the time, introducing visible errors.

---

## Analysis Scenarios

The notebook runs simulations under the following conditions:

### 1. No Noise, No Eavesdropper (Ideal Case)
Baseline run confirming ~0% mismatch rate and ~50% key retention.

### 2. Quantum Channel Noise (No Eavesdropper)
Three Qiskit `NoiseModel` configurations are applied with varying error probabilities `p ∈ [0, 0.5]`:

| Noise Type | Qiskit Gate | Effect |
|---|---|---|
| Bit-flip | `X` error | Randomly flips `|0⟩ ↔ |1⟩` |
| Phase-flip | `Z` error | Randomly flips `|+⟩ ↔ |−⟩` |
| Bit-phase-flip | `Y` error | Combined bit and phase flip |

### 3. Eavesdropping (No Noise)
Eve is added to the circuit. The mismatch rate rises to ~25% due to Eve's random basis choices.

### 4. Eavesdropping + Noise
Both Eve and quantum channel noise are active simultaneously, showing their combined impact.

### 5. Multiple Key Lengths
All scenarios above are repeated for initial key lengths: **50, 100, 300, 700, 1000, 1500 bits**, with `n_repeats=10` runs each and 95% confidence intervals.

---

## Key Metrics

| Metric | Symbol | Description |
|---|---|---|
| Sifted key length | `L` | Number of bits kept after sifting |
| Mismatch ratio | `R_mis` | Fraction of sifted bits where Alice ≠ Bob |
| Keep ratio | `P_keep` | Fraction of initial bits retained (~50%) |
| Eve undetected probability | `P_undetected(k)` | Probability that Eve is not caught when Alice and Bob check a fraction `k` of the sifted key |

The Eve detection probability is computed as:

```
P_undetected(k) = C(L - M, s) / C(L, s)
```

where `L` is the sifted key length, `M = R_mis × L` is the number of erroneous bits, `s = k × L` is the number of checked bits, and `C(n, r)` is the binomial coefficient.

---

## Visualizations

The notebook produces the following plots:

1. **Mismatch & Keep-rate vs. error probability** – single-length curve for each noise type with and without Eve.
2. **Eve detection probability vs. fraction of checked bits** – shows how quickly Alice and Bob can detect an eavesdropper.
3. **Mismatch & Keep-rate grid (2×3)** – the above curves repeated across all six key lengths.
4. **Eve detection probability grid (2×3)** – detection curves across all six key lengths.

All figures are exported as transparent-background PNG files.

---

## References

- Bennett, C. H., & Brassard, G. (1984). *Quantum cryptography: Public key distribution and coin tossing*. Proceedings of the IEEE International Conference on Computers, Systems and Signal Processing, 175–179.
- [Qiskit Documentation](https://docs.quantum.ibm.com/)
- [Qiskit Aer – Noise Simulation](https://qiskit.github.io/qiskit-aer/)
- `BB84-Quantum-Key-Distribution.pdf` (included in this repository)
