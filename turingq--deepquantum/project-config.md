---
trigger: always_on
description: Guidance for AI agents using or modifying this repository.
---

# AGENTS.md

Guidance for AI agents using or modifying this repository.

## Project Overview

This repository is the `deepquantum` Python package. It is a PyTorch-based quantum computing simulation library for quantum circuits, quantum machine learning, photonic quantum computing, measurement-based quantum computation (MBQC), tensor-network/MPS simulation, and distributed simulation.

The core package source is under `src/deepquantum/`. Prefer public APIs exported from `deepquantum` or `deepquantum.photonic` before reaching into private helpers or implementation details.

## Repository Layout

- `src/deepquantum/`: qubit state/circuit simulation, gates, layers, observables, channels, ansatz classes, QASM helpers, circuit cutting, distributed state-vector simulation, and shared math utilities.
- `src/deepquantum/photonic/`: photonic states, `QumodeCircuit`, Fock/Gaussian/Bosonic backends, photonic gates/channels/measurements, Clements/GBS ansatzes, TDM circuits, unitary decomposition/mapping, and photonic distributed simulation.
- `src/deepquantum/mbqc/`: MBQC `Pattern`, `GraphState`, N/E/M/C commands, standardization, signal shifting, and pattern execution.
- `tests/`: source of truth for supported behavior and external compatibility checks.
- `tutorials/`: user-facing notebook tutorials for qubit circuits, photonic circuits, and MBQC.
- `examples/`: runnable notebooks/scripts for algorithms, benchmarks, Clements/unitary mapping, QAOA/HHL/VQE, MBQC, and photonic workflows.
- `docs/`: Sphinx/MyST documentation, quick-start pages, demos, and generated autosummary API pages.

When answering simulation questions, inspect the closest combination of source, tests, tutorial, and example first. Start with `README.md`, then use `tutorials/` or `docs/source/quick_start/`, then match to a nearby `tests/test_*.py` file before writing new code.

## Simulation Usage Guidance

General workflow:

- Import public APIs with `import deepquantum as dq`; use `import deepquantum.photonic as dqp` for photonic-only helpers.
- Start with minimal examples from `README.md`, `tutorials/`, `examples/`, and matching tests. Keep qubit counts, cutoff, bond dimension, batch size, device, and dtype small until correctness is confirmed.
- Use PyTorch tensors for data, parameters, device placement, dtype changes, and autograd. Circuits and states are `torch.nn.Module` objects.
- Prefer public circuit methods such as `cir.h`, `cir.rx`, `cir.cnot`, `cir.observable`, `cir.measure`, `cir.expectation`, `cir.get_prob`, `cir.get_amplitude`, `cir.pattern`, `cir.to(...)`, and photonic equivalents on `QumodeCircuit`.

Qubit circuits:

- Use `dq.QubitCircuit(nqubit, init_state='zeros', den_mat=False, reupload=False, mps=False, chi=None, shots=1024)`.
- State-vector simulation is the default. Call `cir()` or `cir(data=..., state=...)` to run the circuit.
- Add observables with `cir.observable(wires=None, basis='z')`; `basis` supports strings such as `'x'`, `'y'`, `'z'`, or multi-wire strings like `'xy'`.
- Call `cir.expectation()` for exact differentiable expectation values, or `cir.expectation(shots=...)` for sampled estimates.
- Call `cir.measure(shots=..., wires=..., with_prob=True)` after a forward pass for sampled bit-string counts and optional probabilities.
- Use `encode=True` on parameterized gates/layers to consume entries from the `data` tensor. `reupload=True` repeats data when a circuit needs more encoded values than provided.
- `data` may be 1D for one sample or 2D for batch execution; tests verify batched qubit forward, MBQC transpilation, and expectation behavior.
- Density-matrix noise channels are available when `den_mat=True`, including bit flip, phase flip, depolarizing, Pauli, amplitude damping, phase damping, and generalized amplitude damping.

MPS and large qubit simulation:

- Use `dq.QubitCircuit(nqubit, mps=True, chi=...)` for matrix product state simulation. Larger `chi` improves accuracy and increases cost.
- `dq.MatrixProductState(...).full_tensor()` can convert small MPS states back to dense tensors for validation.
- Tests compare MPS against dense simulation for probabilities, amplitudes, and circuit outputs.

Photonic circuits:

- Use `dq.QumodeCircuit(nmode, init_state, cutoff=None, backend='fock', basis=True, den_mat=False, detector='pnrd', mps=False, chi=None, noise=False, mu=0, sigma=0.1)`.
- Fock backend:
  - `basis=True` represents Fock basis states such as `[1, 0, 1]`; `cir(is_prob=None)` returns a unitary, `cir(is_prob=True)` returns probabilities, and `cir(is_prob=False)` returns amplitudes.
  - `basis=False` represents Fock state tensors or superpositions such as `[(amp, [n0, n1])]`.
  - Use `cir.measure(...)`, `cir.get_amplitude(final_state)`, and `cir.get_prob(final_state)`.
  - For Fock MPS, use `backend='fock', basis=False, mps=True, chi=..., cutoff=...`.
- Gaussian backend:
  - Use `backend='gaussian'` with `'vac'` or `[cov, mean]`; output is `[cov, mean]` unless `is_prob=True`.
  - Supported workflows include Gaussian gates (`s`, `s2`, `d`, `r`, `bs`, etc.), `get_symplectic`, `photon_number_mean_var`, Fock probabilities with `detector='pnrd'` or `'threshold'`, and `measure_homodyne`.
- Bosonic backend:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TuringQ/deepquantum](https://github.com/TuringQ/deepquantum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
