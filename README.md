# vqe-quantum-models
VQE ground state estimation for TFIM and Heisenberg models on real IBM quantum hardware

# VQE Quantum Models on Real IBM Hardware

Ground state energy estimation for two fundamental quantum models,
built with Qiskit and tested on a real IBM quantum computer.

---

## What This Project Does

This project implements the Variational Quantum Eigensolver (VQE) algorithm
for two well-known quantum physics models, runs them on real IBM quantum
hardware, and systematically compares different error mitigation strategies.

The goal was simple which take a real quantum model, find its ground state energy
variationally, and honestly measure how far real noisy hardware deviates from
the exact answer. Then try to close that gap with error mitigation.

---

## Models

### Transverse Field Ising Model (TFIM)

H = -J Σ Z_i Z_{i+1} - h Σ X_i

A chain of spins that want to align with each other (ZZ term) competing
against a transverse magnetic field trying to tilt them sideways (X term).
At J = h = 1 the system sits exactly at a quantum phase transition —
the point of maximum entanglement and maximum difficulty for variational
methods. We ran it there deliberately.

### Heisenberg XXX Model

H = J Σ (X_i X_{i+1} + Y_i Y_{i+1} + Z_i Z_{i+1})

Uniform spin-spin exchange coupling in all three spatial directions.
A foundational model in condensed matter physics with SU(2) symmetry.
Better behaved than TFIM for VQE — and our results confirmed exactly that.

Both models use 4 qubits. Small enough to verify against exact diagonalization.
Large enough to show real quantum behavior.

---

## Results

| Method | TFIM Error | Heisenberg Error |
|---|---|---|
| Exact diagonalization (NumPy) | 0.0% | 0.0% |
| VQE — Aer simulator (deep ansatz) | 0.6% | 0.1% |
| VQE — Aer simulator (shallow ansatz) | 3.5% | 5.3% |
| Real IBM hardware — noisy, deep circuit | 24.7% | 25.9% |
| Real IBM hardware — DD, shallow circuit | 9.0% | 10.2% |
| Real IBM hardware — ZNE + DD | 8.9% | 9.7% |
| Real IBM hardware — Twirling + ZNE + DD | 10.2% | 12.2% |
| **Real IBM hardware — PEC + Twirling + DD** | **4.3%** | **6.0%** |

**82% reduction in hardware error** going from the naive baseline (24.7%)
to the best mitigation strategy (4.3%).

![Final Results](results/hardware_comparison_final.png)

---

## Key Findings

**Circuit depth is the dominant noise source.**
Reducing ansatz depth from 148 gates to 18 gates cut hardware error from
24.7% to 9.0% — more than halved. This happened before any error mitigation
was even applied. The lesson: on NISQ devices, a shallower circuit that fits
the hardware beats a deeper circuit that is theoretically more expressive.

**PEC outperforms ZNE significantly here.**
ZNE (resilience_level=1) barely improved results — 9.0% to 8.9%. This is
expected: ZNE assumes incoherent noise but IBM hardware has significant
coherent noise components. PEC (resilience_level=2) with Pauli twirling
converts coherent errors to incoherent first, then cancels them
probabilistically — dropping error to 4.3%.

**Twirling + ZNE is worse than plain ZNE alone.**
Twirling randomizes the noise, which is what PEC needs, but it disrupts
ZNE's extrapolation model. Don't combine twirling with ZNE — use it with
PEC instead.

**TFIM at the critical point is harder than Heisenberg.**
Even on the simulator, TFIM achieved only 3.5% error vs 0.1% for Heisenberg
with the same shallow ansatz. At J = h = 1, TFIM is maximally entangled —
a hardware-efficient ansatz cannot fully represent this state. This is a
known limitation of shallow variational circuits at quantum critical points.

---

## Hardware Details

- **Platform:** IBM Quantum (Open Plan)
- **Backend:** Auto-selected least busy backend with ≥ 4 qubits
- **Shots:** 10,000 per job
- **Error suppression:** Dynamical Decoupling (XpXm sequence)
- **Error mitigation tested:** ZNE, Pauli Twirling, PEC

---

## How to Run

### Simulator only (no IBM account needed)
```bash
pip install qiskit qiskit-aer qiskit-algorithms numpy scipy matplotlib
jupyter notebook
```
Open the notebook and run all cells up to the hardware section.

### Full run including real hardware
1. Create a free account at quantum.cloud.ibm.com
2. Get your API key and CRN from account settings
3. Add them when prompted in the connection cell
4. Note: jobs queue on the free plan — expect 10 min to a few hours wait

---

## Tech Stack

- Qiskit 2.x
- Qiskit Aer 0.17
- Qiskit Algorithms 0.4
- Qiskit IBM Runtime
- NumPy / SciPy / Matplotlib

---

## What I Learned

Starting from scratch on a real quantum hardware experiment teaches you
things no tutorial covers. The queue times are real. The noise is real.
The gap between a clean simulator result and a hardware result is
genuinely humbling and closing that gap systematically using error
mitigation techniques is where the actual research happens.

The 82% error reduction documented here is not a trick. It is the result
of understanding what type of noise is present, choosing the right tool
for that noise type, and being honest about when a technique helps and
when it does not.
