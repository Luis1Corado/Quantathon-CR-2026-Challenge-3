# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A research deliverable for the QWorld Hackathon Quantathon CR 2026 (Challenge 3): quantum
phase transition and quench dynamics studies, each comparing a classical exact-diagonalization
(ED) baseline against quantum approaches (Trotterized time evolution, VQE). There are **two
self-contained notebooks**, one finished and one in progress, plus a hand-derivation theory doc
for the second:

- **`TFIM_ED_Trotter_VQE_merged.ipynb`** — the finished deliverable. **Transverse-Field Ising
  Model** on `N_QUBITS = 6` sites, ED vs. Trotter (`guppylang` + Quantinuum's Selene emulator)
  vs. VQE with a Hardware-Efficient Ansatz. Treat this notebook as the source of truth for the
  TFIM study; `individual_notebooks/` holds earlier, unmerged drafts (classical ED, Trotter/Selene,
  VQE) kept for reference only.
- **`FermiHubbard2D_ED_Trotter_VQE.ipynb`** — a second, newer study of the **2D Fermi-Hubbard
  model on a 4×2 ladder** (8 sites, 16 qubits after Jordan-Wigner), following the same
  ED-vs-Trotter-vs-VQE structure but with a different toolchain: **Qiskit** for circuit
  primitives/simulation/VQE instead of guppylang/Selene. **Sections A (ED baseline) and B
  (Trotterized evolution) are implemented** — Section C (Trotter error analysis), Section D
  (VQE), and Limitations, all described in the theory doc below, are not yet written.
  `requirements.txt` includes `qiskit`/`qiskit-aer`.
- **`fermi_hubbard_theory.md`** — hand-derived theory for the Fermi-Hubbard notebook: lattice
  Hamiltonian, Jordan-Wigner mapping (with the per-bond Z-string costs specific to this 2D
  ladder), Trotter layer structure, and what each planned notebook section should build. Read
  this before extending `FermiHubbard2D_ED_Trotter_VQE.ipynb` — it fixes the qubit ordering,
  layer grouping, and observable definitions the code must match.

`Informe_tecnico/` contains write-ups submitted for the challenge, both in Spanish:
`informe_tecnico_Quantathon_2026_Finalizado (2).pdf` is the final technical report, documenting
the TFIM study; `presentacion_fermi_hubbard.tex`/`.pdf` is a Beamer slide deck covering the
Fermi-Hubbard problem statement and proposed solution (ED/Trotter/VQE plan), companion to
`fermi_hubbard_theory.md` rather than a report on finished results. Rebuild the slides with:

```bash
cd Informe_tecnico && pdflatex presentacion_fermi_hubbard.tex
```

Needs a LaTeX install with `beamer`, `tikz`, `booktabs`, and `braket`.

## Language and docs to keep in sync

Prose language follows the file, and both are in use: `TFIM_ED_Trotter_VQE_merged.ipynb`,
`README.md`, and everything in `Informe_tecnico/` are in **Spanish**; `FermiHubbard2D_ED_Trotter_VQE.ipynb`,
`fermi_hubbard_theory.md`, and this file are in **English**. Match whichever file you are editing
(code identifiers and comments are English throughout).

`README.md` is the user-facing doc and **duplicates TFIM facts that also live here** — the
Hamiltonian convention, `N=6`, the section list, the per-section runtime estimates, the smoke-test
knobs, and the exact list of emitted PNGs. It covers *only* the TFIM study and does not mention
Fermi-Hubbard. If you change any of those TFIM facts, update `README.md` too.

## Physics conventions

**TFIM** (`TFIM_ED_Trotter_VQE_merged.ipynb`):

```
H = -J * Σ_i Z_i Z_{i+1}  -  h * Σ_i X_i        (periodic boundary conditions)
```

`J = 1.0` is fixed; `h/J` is the swept parameter. The critical point is at `h/J = 1`.
`N_QUBITS = 6` is used consistently across ED, Trotter, and VQE sections so results are
directly comparable — do not change `N` in only one section.

**Fermi-Hubbard** (`FermiHubbard2D_ED_Trotter_VQE.ipynb`, see `fermi_hubbard_theory.md` for
full derivation):

```
H = -t * Σ_{⟨i,j⟩,σ} (c†_{i,σ} c_{j,σ} + h.c.)  +  U * Σ_i n_{i,↑} n_{i,↓}  -  μ * Σ_{i,σ} n_{i,σ}
```

4×2 ladder (`x=0..3` periodic/"leg", `y=0,1` open/"rung"), 8 sites, half filling
(`N_↑ = N_↓ = 4`, `μ = U/2`). `t = 1.0` fixed; `U/t` is the swept coupling. This is a 2-leg
ladder, not a true extended 2D lattice — it is **gapped for any `U > 0`** (adiabatically
connected to a rung-singlet Heisenberg ladder), so there is **no finite-`U_c` phase transition**
at half filling; frame results as a **crossover**, diagnosed via double occupancy, charge gap
`Δ_c`, spin gap `Δ_s`, and rung/leg spin correlations — not a sharp critical point. Qubit
ordering is spin-blocked then row-major (`↑` block = positions 0–7, `↓` block = 8–15); nearest-
neighbor lattice bonds are not all JW-adjacent, so some hopping terms carry a Z-string (0, 2, or
3 qubits long depending on bond type — see theory doc §2.3).

## Environment setup

```bash
python3 -m venv quantathon_env
source quantathon_env/bin/activate
pip install -r requirements.txt
```

Requires Python 3.11+ (tested with 3.14). Core dependencies: `numpy`/`scipy`/`matplotlib` for ED
and plotting; `guppylang`/`selene-sim`/`hugr` for quantum circuit construction (TFIM's Trotter,
HEA) and execution on Quantinuum's **Selene** emulator (Quest simulator backend);
`jupyter`/`notebook`/`ipykernel` to open and run the notebooks. All install from precompiled
`manylinux` wheels on Linux — no C/C++ toolchain needed. `qiskit`/`qiskit-aer` are in
`requirements.txt` (used by the Fermi-Hubbard notebook's Section B onward; `qiskit-aer`'s
`AerSimulator` treats `PauliEvolutionGate` as opaque, so circuits are `.decompose(["PauliEvolution"])`'d
before simulation).

Note on the virtualenvs actually present in this checkout (all untracked, none is
`quantathon_env` — that has to be created): `venv/` has Python 3.12 with only
`numpy`/`scipy`/`matplotlib`/`jupyter` installed, so it can run the ED sections of either
notebook, plus Fermi-Hubbard's Section B (`qiskit`/`qiskit-aer` were installed into it ad hoc to
develop and verify that section) — but **not** TFIM's Selene/guppylang sections;
`quantum_phase_transition/` has no dependencies at all. The system `python3` is missing
`packaging`, so `import matplotlib` fails outside a venv. Before running TFIM's circuit code,
build `quantathon_env` from `requirements.txt` (or add `guppylang`/`selene-sim`/`hugr` to `venv/`).

## Running the notebooks

```bash
source quantathon_env/bin/activate
jupyter notebook TFIM_ED_Trotter_VQE_merged.ipynb        # or FermiHubbard2D_ED_Trotter_VQE.ipynb
# or non-interactively:
jupyter nbconvert --to notebook --execute --inplace TFIM_ED_Trotter_VQE_merged.ipynb
```

Running `TFIM_ED_Trotter_VQE_merged.ipynb` regenerates the figures in the repo root:
`tfim_phase_diagram.png`, `tfim_quench_h{0.5,1.0,2.0}_trotter_vs_exact.png`,
`trotter_error_convergence.png`, `tfim_vqe_vs_ed_N6.png`.

Running `FermiHubbard2D_ED_Trotter_VQE.ipynb` (Sections A–B) regenerates
`fermi_hubbard_2d_phase_diagram.png` and `fermi_hubbard_quench_U{1.0,4.0,8.0}_trotter_vs_exact.png`.

**Expected runtime is dominated by circuit simulation, not ED**, for the TFIM notebook. Section
A (ED, N=6) runs in seconds. Sections B/C/D compile and run real circuits on Selene (2000
shots/circuit each): Section B (quench sweep, 3 values of h/J) ~10 min; Section C (Trotter
convergence sweep) a couple minutes; Section D (VQE, COBYLA over ~11 h/J values) is the most
expensive — potentially tens of minutes, since each optimizer step recompiles and re-runs the
circuit. For a quick smoke test, temporarily lower `N_SHOTS`, `STEP_COUNTS`/`N_STEPS_SCAN`
(Sections B/C), or `maxiter` in the `run_vqe` call (Section D) before running the full notebook.

The Fermi-Hubbard notebook's ED baseline is also fast (whole `U/t` sweep in seconds) **because
it never touches the full `2^16` space**: it diagonalizes inside fixed `(N_↑, N_↓)` sectors, and
the half-filling sector is only `C(8,4)^2 = 4900`-dimensional. The `2^16 = 65536` figure in the
theory doc (§2.5) is the cost the *circuit* sections will pay, not Section A's — so once
Trotter/VQE are added, shot counts and optimizer `maxiter` will need at least the same budgeting
as the TFIM notebook, likely more. For a quick smoke test of Section A, shorten
`U_OVER_T_VALUES` (currently 17 points over `U/t ∈ [0, 8]`).

Section B (16-qubit Trotter circuit, Qiskit Aer statevector simulation, 2000 shots/circuit) runs
in **under a minute**: `U_QUENCH_VALUES = [1.0, 4.0, 8.0]`, each swept over `STEP_COUNTS` (26
points up to `N_STEPS=50`), took ~15s per `U/t` value when last run — much faster than TFIM's
Selene-based Section B, since Aer's statevector method has far less per-circuit overhead than
compiling to and executing on Selene. For a quick smoke test, lower `N_SHOTS` or thin out
`STEP_COUNTS`.

There is **no test suite, linter, or CI** in this repo — the deliverables are the notebooks, the
figures they emit, and the documents in `Informe_tecnico/`. "Verifying a change" means re-running
the affected notebook section (or `nbconvert --execute`) and checking the regenerated figure
against the physics expectations stated in the theory/limitations cells.

## Notebook architecture

### `TFIM_ED_Trotter_VQE_merged.ipynb` (complete)

Organized into sections, each building on shared helpers from earlier cells — when editing one
section, check whether it reuses functions/constants defined upstream:

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

Reproducibility notes: `N=6` is shared across ED, Trotter, and VQE — a deliberate choice for
direct comparability. The Section B Trotter step `Δt = T_MAX / N_STEPS` is reused (with its own
sweep) in Section C's error analysis — don't let these drift independently. VQE uses `seed=0`;
results are not guaranteed globally optimal (COBYLA is local).

### `FermiHubbard2D_ED_Trotter_VQE.ipynb` (in progress)

Mirrors the TFIM notebook's section structure (see `fermi_hubbard_theory.md` §3 for the full
plan); Sections A and B are implemented, C/D/Limitations are not:

- **Section A — Exact diagonalization (classical baseline)**, implemented: `basis_states`
  (Fock basis for a fixed `(N_↑, N_↓)` particle-number sector), `build_fermi_hubbard_operators`
  (hopping + interaction + number operators as sparse matrices on that sector), ground state and
  observables (double occupancy, rung/leg spin correlations) at half filling, then a crossover
  sweep over `U/t` computing double occupancy, charge gap `Δ_c`, and spin gap `Δ_s` (the latter
  two require diagonalizing adjacent particle-number/spin sectors, not just the half-filling
  ground state). All plain numpy/scipy — no Qiskit used in Section A.
  - Two invariants to preserve when extending it. **(i) Sector operators are built once, not per
    `U`**: `build_fermi_hubbard_operators` returns `(hop, n_int, n_tot, basis, index)` whose
    structure is `U`-independent, so cell A.5 precomputes four sectors into module-level globals
    (`HOP_HALF`/`NINT_HALF`/…, plus `*_NP1`, `*_NM1`, `*_SZ1` for the gaps) and the sweep only
    reassembles `H = -t·hop + diag(U·n_int − μ·n_tot)` via `fermi_hubbard_hamiltonian`. New
    observables should reuse those globals rather than rebuild a sector. Note `μ = U/2` is set
    inside the sweep loop, not as a global constant.
  - **(ii) The ED fermionic sign convention is the JW convention.** Fock states are integers with
    the `↑` block in bits 0–7 and the `↓` block in bits 8–15, and `apply_hop` computes the
    `c†_p c_q` sign as the parity of occupied modes below each index — i.e. exactly the
    `Π_{k<j} Z_k` string of the theory doc §2.1, under the same qubit ordering. This is what lets
    Section A's numbers be compared term-by-term against the future Qiskit circuits; don't
    reorder the blocks or change the sign rule in one place only.
- **Section B — Trotterized evolution (Qiskit)**, implemented: the 7 mutually-commuting Trotter
  layers from theory doc §2.4 (leg-even, leg-odd, rung hopping per spin, plus one interaction
  layer), each built from hand-specified Pauli terms (`jw_hop_pauli_terms`, dressed with the
  Z-strings from §2.3) wrapped in Qiskit's `PauliEvolutionGate` and run on Qiskit Aer
  (`AerSimulator`, statevector method, 2000 shots) — hand-derived Hamiltonian/mapping/Trotter
  step, Qiskit only for circuit primitives and simulation, no auto-mapper.
  - **Exact factorization, not an approximation, at the per-bond level.** A hopping bond's dressed
    `XX_str` and `YY_str` terms commute, so `append_hop_gate` applies `e^{-iΔt H_bond}` as two
    sequential `PauliEvolutionGate`s with no Trotter error of its own — the only Trotter error is
    from the 7 layers not commuting with each other. At half filling (`μ=U/2`), the interaction
    term's single-qubit `Z` pieces cancel exactly against the chemical-potential term, leaving a
    pure `(U/4)·Z_↑Z_↓` rotation per site (`append_interaction_gate`) plus a dropped global phase
    — both identities are checked numerically against `apply_hop`'s fermionic sign convention in
    the section's own derivation, not just asserted.
  - **Quench protocol: doublon decay, not a `h/J`-style field quench.** There's no swept field
    analogous to TFIM's `h`, so the quench is an interaction quench: start in a charge-density-wave
    Fock state (leg `y=0` fully doubly occupied, `y=1` empty — same half-filling sector as Section
    A) and evolve under fixed `U/t ∈ {1, 4, 8}`, watching double occupancy relax and charge
    redistribute across rungs (`doublon_decay_scan` for the circuit, `doublon_decay_exact` for the
    sector-restricted ED reference via `expm_multiply` — both conserve `(N_↑,N_↓)` exactly, so the
    comparison never leaves the 4900-dim half-filling sector built in Section A). This is the
    standard "doublon decay" setup from cold-atom Hubbard experiments (e.g. Strohmaier et al.
    2010); larger `U/t` visibly suppresses the decay — the dynamical counterpart to Section A's
    crossover, not a new physical claim.
  - `PauliEvolutionGate` is opaque to Aer — every circuit is `.decompose(["PauliEvolution"])`'d
    before `sim.run(...)`, or the run raises `AerError: unknown instruction: PauliEvolution`.
- **Section C — Trotter error analysis, not yet implemented**: same convergence-vs-Δt structure
  as the TFIM notebook's Section C, adapted to this Hamiltonian.
- **Section D — VQE, not yet implemented**: planned ansatz should conserve particle number and
  ideally total spin (a hopping-gate/Givens-rotation ansatz built from the same JW mapping, per
  theory doc §3) rather than a generic hardware-efficient ansatz, which would explore unphysical
  number/spin sectors.
- **Limitations**, not yet written: should carry over the TFIM notebook's caveats plus the
  ladder-specific ones from the theory doc — this is a crossover study (no finite-`U_c`
  transition, §1.3), and 2D-vs-1D JW mapping adds Z-strings/CNOT overhead not present in TFIM.
