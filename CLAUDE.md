# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A research deliverable for the QWorld Hackathon Quantathon CR 2026 (Challenge 3): a study of
the **Transverse-Field Ising Model (TFIM)** quantum phase transition and quench dynamics,
comparing a classical exact-diagonalization (ED) baseline against two quantum approaches —
Trotterized time evolution and VQE with a Hardware-Efficient Ansatz (HEA).

The entire deliverable is a single self-contained notebook:

```
TFIM_ED_Trotter_VQE_merged.ipynb
```

There is no separate application code, package, or test suite — all logic (physics, circuit
construction, simulation, plotting) lives in this notebook's cells. `individual_notebooks/`
holds earlier, unmerged drafts (classical ED, Trotter/Selene, VQE) kept for reference only —
treat `TFIM_ED_Trotter_VQE_merged.ipynb` as the single source of truth.

## Physics convention

```
H = -J * Σ_i Z_i Z_{i+1}  -  h * Σ_i X_i        (periodic boundary conditions)
```

`J = 1.0` is fixed; `h/J` is the swept parameter. The critical point is at `h/J = 1`.
`N_QUBITS = 6` is used consistently across ED, Trotter, and VQE sections so results are
directly comparable — do not change `N` in only one section.

## Environment setup

```bash
python3 -m venv quantathon_env
source quantathon_env/bin/activate
pip install -r requirements.txt
```

Requires Python 3.11+. Core dependencies: `numpy`/`scipy`/`matplotlib` for ED and plotting;
`guppylang`/`selene-sim`/`hugr` for quantum circuit construction (Trotter, HEA) and execution
on Quantinuum's **Selene** emulator (Quest simulator backend). All install from precompiled
`manylinux` wheels — no C/C++ toolchain needed on Linux.

Note: `quantum_phase_transition/` and `venv/` in the repo root are local, untracked virtualenvs
— not part of the project structure.

## Running the notebook

```bash
source quantathon_env/bin/activate
jupyter notebook TFIM_ED_Trotter_VQE_merged.ipynb    # interactive, Run All
# or non-interactively:
jupyter nbconvert --to notebook --execute --inplace TFIM_ED_Trotter_VQE_merged.ipynb
```

Running it regenerates the figures in the repo root: `tfim_phase_diagram.png`,
`tfim_quench_h{0.5,1.0,2.0}_trotter_vs_exact.png`, `trotter_error_convergence.png`,
`tfim_vqe_vs_ed_N6.png`.

**Expected runtime is dominated by circuit simulation, not ED.** Section A (ED, N=6) runs in
seconds. Sections B/C/D compile and run real circuits on Selene (2000 shots/circuit each):
Section B (quench sweep, 3 values of h/J) ~10 min; Section C (Trotter convergence sweep) a
couple minutes; Section D (VQE, COBYLA over ~11 h/J values) is the most expensive — potentially
tens of minutes, since each optimizer step recompiles and re-runs the circuit. For a quick
smoke test, temporarily lower `N_SHOTS`, `STEP_COUNTS`/`N_STEPS_SCAN` (Sections B/C), or
`maxiter` in the `run_vqe` call (Section D) before running the full notebook.

## Notebook architecture

The notebook is organized into sections, each building on shared helpers from earlier cells —
when editing one section, check whether it reuses functions/constants defined upstream:

- **Theory** — defines the Hamiltonian and the observables reported throughout: `⟨Z⟩`, `⟨X⟩`,
  `⟨Z_i Z_{i+1}⟩` (nearest-neighbor correlation).
- **Section A — Exact diagonalization (classical baseline)**: `build_tfim_hamiltonian`,
  `analyze_tfim` (phase diagram sweep over `h/J`), `time_evolve_exact` (exact quench dynamics,
  used as the reference curve in Section B).
- **Section B — Trotterized evolution (guppylang + Selene)**: `make_circuit`/`run_circuit`
  build and execute the Trotter-step circuit locally on Selene's Quest simulator; quench
  dynamics for `h/J ∈ {0.5, 1.0, 2.0}` compared against Section A's ED reference. Circuit
  parameters are baked in as closure constants and the circuit is recompiled per evaluation —
  this pattern repeats in Section D's HEA circuit.
  - Has a commented-out cloud appendix for execution on Quantinuum Nexus (`qnx.login()`,
    interactive, requires a Nexus account) — not needed to reproduce results, left disabled by
    default.
- **Section C — Trotter error analysis**: `trotter_error_at_h` / `make_circuit_dt` sweep the
  Trotter step size `Δt = T_MAX / N_STEPS` at the critical point and measure convergence
  against the exact reference.
- **Section D — VQE with a Hardware-Efficient Ansatz**: `make_hea_circuit` (ansatz),
  `evaluate_energy_local` (runs both Z- and X-basis circuits on Selene to estimate energy),
  `exact_ground_state_observables` (ED reference reused from Section A) — sweeps `h/J` and
  compares VQE-found ground states against ED. Optimization uses `scipy.optimize.minimize`
  (COBYLA) with a fixed seed (`seed=0`) for parameter initialization; as a local optimizer,
  COBYLA has no global-minimum guarantee.
  - Also has a disabled-by-default Quantinuum Nexus cloud appendix, same caveats as Section B.
- **Limitations** — an explicit, honest section on finite-size effects, Trotter error, shot
  noise, ideal-simulator-vs-real-hardware gap, VQE non-convergence, and boundary convention.
  Keep this section in sync with reality if changing `N`, shot counts, or optimizer settings.

## Reproducibility notes

- `N=6` is shared across ED, Trotter, and VQE — a deliberate choice for direct comparability.
- The Section B Trotter step `Δt = T_MAX / N_STEPS` is reused (with its own sweep) in Section
  C's error analysis — don't let these drift independently.
- VQE uses `seed=0`; results are not guaranteed globally optimal (COBYLA is local).
