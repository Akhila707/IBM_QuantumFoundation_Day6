# IBM Quantum Foundations — Day 6

<div align="center">

![IBM Quantum](https://img.shields.io/badge/IBM%20Quantum-052FAD?style=flat-square&logo=ibm&logoColor=white)
![Qiskit](https://img.shields.io/badge/Qiskit-6929C4?style=flat-square&logoColor=white)
![Python 3.10](https://img.shields.io/badge/Python%203.10-1a1a2e?style=flat-square&logo=python&logoColor=4fc3f7)
![ZNE](https://img.shields.io/badge/ZNE-Error%20Mitigation-4fc3f7?style=flat-square)
![Day 6](https://img.shields.io/badge/Day%2006-Complete-4fc3f7?style=flat-square)
![Day 7](https://img.shields.io/badge/Day%2007-Loading...-555555?style=flat-square)

</div>

<br/>

```
 
                                                               
      noise ×1  →  P(|00⟩) = 0.4980                              
      noise ×2  →  P(|00⟩) = 0.4630                              
      noise ×3  →  P(|00⟩) = 0.4620                              
                            ↓                                   
      ZNE mitigated  →  P(|00⟩) = 0.5103  (true = 0.5000)        
                                                                
  
```

<div align="center">
<i>Part of the IBM Quantum 20-Day Learning Sprint · VIT Chennai</i>
</div>

---
```
## Overview

This notebook implements **Zero-Noise Extrapolation (ZNE)** — a quantum error mitigation technique that recovers accurate results from noisy quantum hardware.

Real quantum computers make mistakes. Gates are imperfect. Qubits lose their state. Measurements drift. ZNE doesn't fix the hardware — it works around the noise mathematically.

| Component | Role |
|-----------|------|
| Bell State circuit | Base circuit to test noise effects |
| Depolarizing noise model | Simulates real IBM hardware errors |
| Noise amplification (X·X pairs) | Intentionally scales noise by ×1, ×2, ×3 |
| Linear extrapolation | Fits a line and projects back to zero noise |
| AerSimulator | Local simulation — 1000 shots per run |
```
---
```
## The Problem — Why Noise Matters

Days 1 through 5 used AerSimulator in perfect mode. No errors. Clean results every time.

Real IBM quantum hardware is different.
```
```
What goes wrong on real hardware:

Gate errors    →  every gate has ~0.1–1% chance of failure
Decoherence    →  qubits lose their quantum state over time
Readout errors →  measuring a qubit gives the wrong answer
Crosstalk      →  nearby qubits interfere with each other
```

A Bell State should always give exactly `{'00': 500, '11': 500}` across 1000 shots. On noisy hardware, you get something like `{'00': 480, '11': 480, '01': 20, '10': 20}`. Those 40 wrong answers are noise.

Error mitigation is the set of techniques that reduce this effect — not by fixing the hardware, but by being clever about how we run and interpret circuits.

---

## The Solution — Zero-Noise Extrapolation

ZNE is based on a counterintuitive idea: to get a result with less noise, first get results with more noise.

```
The thermometer analogy:

Your thermometer always reads 2° too high.
You cannot recalibrate it.

But you can observe a pattern:
  thermometer at setting 1  →  reads 22°  (true + 2)
  thermometer at setting 2  →  reads 24°  (true + 4)
  thermometer at setting 3  →  reads 26°  (true + 6)

Draw a line through (1, 22), (2, 24), (3, 26).
Extend it back to setting 0.
It crosses at 20° — the true temperature.

ZNE does exactly this with quantum circuits.
```

---

## How Noise Gets Amplified

To run the circuit at higher noise levels, we insert gate pairs that cancel mathematically but add noise physically.

```
X · X = Identity

Mathematically: applying X twice returns to the original state.
Physically: each X gate on real hardware introduces error.

noise ×1  →  run the original circuit
noise ×2  →  insert one X·X pair after each gate
noise ×3  →  insert two X·X pairs after each gate

The quantum state is unchanged in theory.
The noise accumulates in practice.
```

---

## Algorithm

```
Step 1  →  Build Bell State circuit
           H → CNOT → Measure
           True result: P(|00⟩) = 0.5000

Step 2  →  Add depolarizing noise model
           Single qubit error rate : 1%
           Two qubit (CX) error rate: 10%

Step 3  →  Run at three noise levels
           noise ×1  →  measure P(|00⟩)
           noise ×2  →  measure P(|00⟩)
           noise ×3  →  measure P(|00⟩)

Step 4  →  Fit a line through the three points
           Extrapolate to noise = 0
           Read off the mitigated result
```

---

## Results

```
Measured values:
  noise ×1  →  P(|00⟩) = 0.4980
  noise ×2  →  P(|00⟩) = 0.4630
  noise ×3  →  P(|00⟩) = 0.4620

After ZNE extrapolation:
  ZNE mitigated  →  P(|00⟩) = 0.5103
  True value     →  P(|00⟩) = 0.5000

Comparison:
  Raw noisy result : 0.4980  (error = 0.0020 from truth)
  ZNE mitigated    : 0.5103  (overshoots slightly)
```

The extrapolated result crosses back past the true value — this is expected with linear fitting on non-linear noise. A polynomial fit would give a closer result. The key point: ZNE moves the estimate back toward the true value, which raw measurement cannot do.

---

## Convergence Plot

```
P(|00⟩) vs noise scale factor:

0.54 ┤
     │ ● ← ZNE mitigated (0.5103)
0.52 ┤
0.50 ┤────────────────────────────── true value (0.5000)
     │  ●  ← noise ×1 (0.4980)
0.48 ┤
0.46 ┤         ●  ← noise ×2 (0.4630)
     │                  ●  ← noise ×3 (0.4620)
0.44 ┤
     └──────────────────────────────────────────
     0    1         2         3
          noise scale factor
```

The dashed line is the linear fit. Extending it left to scale = 0 gives the mitigated estimate.

---

## Error Types Covered

| Error type | Cause | Effect |
|------------|-------|--------|
| Depolarizing error | Gate imperfection | Random bit/phase flips |
| Decoherence | Environmental interference | Qubit loses quantum state |
| Readout error | Measurement drift | Wrong classical bit recorded |
| Crosstalk | Qubit proximity | Unintended qubit interactions |

---

## Tech Stack

```python
qiskit              >= 1.0.0    # quantum circuit construction
qiskit-aer          >= 0.17.2   # local simulation + noise models
qiskit-aer.noise    >= 0.17.2   # depolarizing error models
numpy               >= 1.24.0   # extrapolation + curve fitting
matplotlib          >= 3.7.0    # result visualization
python-dotenv       >= 1.0.0    # credential isolation
```

---

## Setup

```bash
git clone https://github.com/Akhila707/IBM_QuantumFoundation_Day6.git
cd IBM_QuantumFoundation_Day6
pip install -r requirements.txt
jupyter notebook
```

---

## Project Structure

```
IBM_QuantumFoundation_Day6/
│
├── notebooks/
│   └── 01_error_mitigation.ipynb
│
├── results/
│   ├── zne_results.png
│   └── noise_comparison.png
│
├── .env
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Sprint Progress

```
Day 01  ──  ✅  Qiskit setup · Hello Quantum · first IBM cloud circuit
Day 02  ──  ✅  Superposition · Entanglement · Multi-gate circuits
Day 03  ──  ✅  Gates deep-dive · Grover's algorithm
Day 04  ──  ✅  VQE · parametric circuits · COBYLA optimizer
Day 05  ──  ✅  QAOA · MaxCut · optimal partition found
Day 06  ──  ✅  Quantum Error Mitigation · noise models · ZNE
Day 07  ──  ⬡   Run all experiments on real IBM hardware
```
---

## What's Next — Day 7

Day 7 is the milestone of the first week.

Every experiment from Days 1 through 6 runs again — this time on real IBM quantum hardware. Not a simulator. Real qubits, real noise, real results.

The comparison between simulated and hardware results is what makes this portfolio piece genuinely different from anything built purely in a local environment.

---

## Security

- Credentials stored in `.env`, excluded from version control via `.gitignore`
- All results in this repo are reproducible locally using AerSimulator — no IBM token required

---

<div align="center">

[![GitHub](https://img.shields.io/badge/Akhila707-181717?style=flat-square&logo=github)](https://github.com/Akhila707)
&nbsp;·&nbsp;
[![IBM Quantum](https://img.shields.io/badge/IBM%20Quantum-052FAD?style=flat-square&logo=ibm&logoColor=white)](https://quantum.ibm.com)

</div>
