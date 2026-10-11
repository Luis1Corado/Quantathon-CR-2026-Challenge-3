# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A research deliverable for the QWorld Hackathon Quantathon CR 2026 (Challenge 3): quantum
phase transition and quench dynamics studies, each comparing a classical exact-diagonalization
(ED) baseline against quantum approaches (Trotterized time evolution, VQE). There are **two
self-contained notebooks**, both finished, plus a hand-derivation theory doc for the second:

- **`TFIM_ED_Trotter_VQE_merged.ipynb`** — the finished deliverable. **Transverse-Field Ising
  Model** on `N_QUBITS = 6` sites, ED vs. Trotter (`guppylang` + Quantinuum's Selene emulator)
  vs. VQE with a Hardware-Efficient Ansatz. Treat this notebook as the source of truth for the
  TFIM study; `individual_notebooks/` holds earlier, unmerged drafts (classical ED, Trotter/Selene,
  VQE) kept for reference only.
- **`FermiHubbard2D_ED_Trotter_VQE.ipynb`** — a second study of the **2D Fermi-Hubbard model on
  a 3×2 ladder** (6 sites, 12 qubits after Jordan-Wigner), following the same
  ED-vs-Trotter-vs-VQE structure but with a different toolchain: **Qiskit** for circuit
  primitives/simulation/VQE instead of guppylang/Selene. **All sections are implemented**:
  A (ED baseline), B (Trotterized evolution), C (Trotter error analysis), D (VQE), and
  Limitations. `requirements.txt` includes `qiskit`/`qiskit-aer`. **The 3×2 size was not the
  original plan** — the lattice was originally 4×2 (8 sites, 16 qubits); it was reduced for the
  *whole notebook* once Section D's VQE forced the issue (see the Notebook architecture section
  below, and the notebook's own intro / Section D for the full empirical account). A structural
  consequence worth knowing up front: a 3-site ring (unlike 4×2's 4-site ring) is **not
  bipartite**, so Section B/C's Trotter circuit needs **9** mutually-commuting layers, not 7.
- **`fermi_hubbard_theory.md`** — hand-derived theory for the Fermi-Hubbard notebook: lattice
  Hamiltonian, Jordan-Wigner mapping (with the per-bond Z-string costs specific to this 2D
  ladder), Trotter layer structure, and what each notebook section builds. Read this before
  extending `FermiHubbard2D_ED_Trotter_VQE.ipynb` — it fixes the qubit ordering, layer grouping,
  and observable definitions the code must match. Describes the **3×2** lattice (updated to match
  the notebook); it records *why* it's 3×2 in §1.2 but, being a physics/mapping document, points
  to `CLAUDE.md`/the notebook for the optimizer-level empirical account rather than repeating it.

`Informe_tecnico/` contains write-ups submitted for the challenge, both in Spanish:
`informe_tecnico_Quantathon_2026_Finalizado (2).pdf` is the final technical report, documenting
the TFIM study; `presentacion_fermi_hubbard.tex`/`.pdf` is a Beamer slide deck covering the
Fermi-Hubbard problem statement and proposed solution (ED/Trotter/VQE plan at the originally
planned 4×2 size — written before the 3×2 reduction, so it's now stale on lattice size; it's a
companion to the theory doc's plan, not a report on finished results, and wasn't updated when
the notebook changed size); `presentacion_fermi_hubbard_resultados.tex`/`.pdf` is its follow-up
— same Beamer theme/palette, but covering the *results* now that the notebook is complete (one
slide per section's actual figure/finding, the 4×2→3×2 change explained up front, Limitations
summarized at the end). Numbers quoted in it are a snapshot from when it was written; if a
notebook re-run changes them meaningfully, update the slide text, not just the embedded figures
(the figures auto-update on rebuild since they're `\includegraphics`'d from the repo root, but
numbers typed directly into the slide text do not). Rebuild either with:

```bash
cd Informe_tecnico && pdflatex presentacion_fermi_hubbard.tex                    # or _resultados.tex
```

Needs a LaTeX install with `beamer`, `tikz`, `booktabs`, and `braket`. `presentacion_fermi_hubbard_resultados.tex`
also needs `graphicx` (for the embedded PNGs) and must be built from inside `Informe_tecnico/` —
it references the root-level `fermi_hubbard_*.png` figures as `../fermi_hubbard_*.png`.

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

3×2 ladder (`x=0..2` periodic/"leg", `y=0,1` open/"rung"), 6 sites, half filling
(`N_↑ = N_↓ = 3`, `μ = U/2`). `t = 1.0` fixed; `U/t` is the swept coupling. This is a 2-leg
ladder, not a true extended 2D lattice — it is **gapped for any `U > 0`** (adiabatically
connected to a rung-singlet Heisenberg ladder), so there is **no finite-`U_c` phase transition**
at half filling; frame results as a **crossover**, diagnosed via double occupancy, charge gap
`Δ_c`, spin gap `Δ_s`, and rung/leg spin correlations — not a sharp critical point. Qubit
ordering is spin-blocked then row-major (`↑` block = positions 0–5, `↓` block = 6–11); nearest-
neighbor lattice bonds are not all JW-adjacent, so some hopping terms carry a Z-string (0, 1, or
2 qubits long depending on bond type — see theory doc §2.3). The lattice size (`N=6`, not the
originally planned 8) is consistent across every section of this notebook, same spirit as TFIM's
`N=6` convention — do not change it in only one section.

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
before simulation; Sections C/D instead read exact expectation values directly from
`qiskit.quantum_info.Statevector`, bypassing `AerSimulator`/shots entirely — faster and simpler
than the `primitives.Estimator` wrapper at this circuit size, see the Notebook architecture
section).

Note on the virtualenvs actually present in this checkout (all untracked, none is
`quantathon_env` — that has to be created): `venv/` has Python 3.12 with
`numpy`/`scipy`/`matplotlib`/`jupyter` plus `qiskit`/`qiskit-aer` installed (added ad hoc to
develop and verify the Fermi-Hubbard notebook), so it can run the ED sections of either notebook
plus the entire Fermi-Hubbard notebook — but **not** TFIM's Selene/guppylang sections;
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

Running `FermiHubbard2D_ED_Trotter_VQE.ipynb` (all sections) regenerates
`fermi_hubbard_2d_phase_diagram.png`, `fermi_hubbard_quench_U{1.0,4.0,8.0}_trotter_vs_exact.png`,
`fermi_hubbard_trotter_error_convergence.png`, and `fermi_hubbard_vqe_vs_ed.png`.

**Expected runtime is dominated by circuit simulation, not ED**, for the TFIM notebook. Section
A (ED, N=6) runs in seconds. Sections B/C/D compile and run real circuits on Selene (2000
shots/circuit each): Section B (quench sweep, 3 values of h/J) ~10 min; Section C (Trotter
convergence sweep) a couple minutes; Section D (VQE, COBYLA over ~11 h/J values) is the most
expensive — potentially tens of minutes, since each optimizer step recompiles and re-runs the
circuit. For a quick smoke test, temporarily lower `N_SHOTS`, `STEP_COUNTS`/`N_STEPS_SCAN`
(Sections B/C), or `maxiter` in the `run_vqe` call (Section D) before running the full notebook.

**The Fermi-Hubbard notebook, end to end (all 4 sections), takes well under 10 minutes** —
dramatically lighter than TFIM despite covering the same A/B/C/D structure, both because the
lattice is small (12 qubits) and because Qiskit Aer's statevector method has far less
per-circuit overhead than compiling to and executing on Selene:

- **Section A** (ED): whole `U/t` sweep (17 points) in under a second — it never touches the
  full `2^12` space, only the fixed-`(N_↑,N_↓)` sectors (`C(6,3)^2 = 400`-dimensional at half
  filling). For a quick smoke test, shorten `U_OVER_T_VALUES`.
- **Section B** (Trotter quench, 2000 shots/circuit): ~10s per `U/t` value ×
  `U_QUENCH_VALUES = [1.0, 4.0, 8.0]`, each swept over `STEP_COUNTS` (26 points up to
  `N_STEPS=50`) — under a minute total. For a quick smoke test, lower `N_SHOTS` or thin out
  `STEP_COUNTS`.
- **Section C** (Trotter error, exact shot-free statevector evaluation): ~40s total,
  `N_STEPS_SCAN = [5, 10, 20, 40, 80, 160]` at `U_FOR_ERROR_SCAN=4.0`, dominated by the
  `n_steps=160` point (circuit depth scales with `n_steps`, and `Statevector.from_instruction`
  cost scales with that depth). For a quick smoke test, drop the largest `n_steps` values.
- **Section D** (VQE, exact shot-free statevector evaluation): the most expensive section at
  ~75s per `U/t` value × 4 values (`U_VQE_SWEEP_VALUES = [0.0, 2.0, 4.0, 8.0]`) — ~5 min total
  — each running 3 COBYLA restarts of `maxiter=600` at `N_LAYERS_VQE=2` (48 parameters). For a
  quick smoke test, lower `n_restarts`/`maxiter` in `run_vqe` or shorten
  `U_VQE_SWEEP_VALUES`.

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

### `FermiHubbard2D_ED_Trotter_VQE.ipynb` (complete)

Mirrors the TFIM notebook's section structure (see `fermi_hubbard_theory.md` §3 for the plan).
All sections (A, B, C, D, Limitations) are implemented, on a single **3×2 lattice (6 sites, 12
qubits)** used consistently throughout — unlike an earlier phase of this notebook, there is no
longer a per-section lattice-size split; everything below reuses the same global
`L`/`LX`/`LY`/`NQ`/`BONDS`/`site_index` defined in Section A.1.

- **Why 3×2, not the originally-planned 4×2 (8 sites, 16 qubits).** ED and Trotter alone would
  have been fine at 4×2; **Section D's VQE is what forced the reduction**. A Hamiltonian
  Variational Ansatz at 16 qubits, with one shared angle per Trotter-style layer (7
  params/layer) and COBYLA (matching TFIM's Section D optimizer), did not converge even at the
  easy `U=0` free-fermion sanity check: 1–6 layers and 150–300 COBYLA iterations left a large,
  non-shrinking gap (best found: `-5.28` vs. the exact `-12.00`). A gradient-based optimizer
  (justified since evaluation is exact/noiseless) helped only marginally and cost
  `2×(#params)` circuit evaluations per step — one run took 24 minutes and still didn't close
  the gap. This was verified as a genuine cost/convergence trade-off, not a bug, by
  hand-checking the Hamiltonian/circuit against the ED reference on the bare, unrotated
  reference state's energy before concluding the ansatz/optimizer combination was the
  bottleneck. At 12 qubits each circuit evaluation is only ~0.04s (vs. ~0.3–0.7s at 16), which
  made **one free angle per bond** (more expressive than per-layer sharing) and **multiple
  COBYLA random restarts** per `U/t` affordable — enough to close *most* (not all) of the gap.
  Given this, the whole notebook (not just Section D) was switched to the smaller lattice,
  rather than run VQE on a different system than ED/Trotter — see the notebook's own intro and
  Section D's markdown for the fuller telling, and `fermi_hubbard_theory.md` §1.2/§2.4/§2.5 for
  how the lattice-specific numbers (bond counts, Z-string lengths, Trotter layer count) changed.
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
    inside the sweep loop, not as a global constant. **Section D now also reuses these same
    `HOP_HALF`/`NINT_HALF`/`NTOT_HALF`/`BASIS_HALF` globals directly** for its ED comparison —
    there's only one ED reference in this notebook now, not a separate one per section.
  - **(ii) The ED fermionic sign convention is the JW convention.** Fock states are integers with
    the `↑` block in bits 0–5 and the `↓` block in bits 6–11, and `apply_hop` computes the
    `c†_p c_q` sign as the parity of occupied modes below each index — i.e. exactly the
    `Π_{k<j} Z_k` string of the theory doc §2.1, under the same qubit ordering. This is what lets
    Section A's numbers be compared term-by-term against the Qiskit circuits in Sections B–D;
    don't reorder the blocks or change the sign rule in one place only.
- **Section B — Trotterized evolution (Qiskit)**, implemented: **9** mutually-commuting Trotter
  layers (theory doc §2.4 — more than a 4×2 lattice's 7, because a 3-site ring is **not
  bipartite**: unlike a 4-site ring's clean "even"/"odd" 2-coloring, any 2 of a triangle's 3
  edges share a vertex, so each edge *position* needs its own layer; `leg_bond_layers`
  implements this by pairing each row's edge `k` with the other row's edge `k`, since those
  *are* vertex-disjoint). Each layer is built from hand-specified Pauli terms
  (`jw_hop_pauli_terms`, dressed with the Z-strings from §2.3) wrapped in Qiskit's
  `PauliEvolutionGate` and run on Qiskit Aer (`AerSimulator`, statevector method, 2000 shots) —
  hand-derived Hamiltonian/mapping/Trotter step, Qiskit only for circuit primitives and
  simulation, no auto-mapper.
  - **Exact factorization, not an approximation, at the per-bond level.** A hopping bond's
    dressed `XX_str` and `YY_str` terms commute, so `append_hop_gate` applies `e^{-iΔt H_bond}`
    as two sequential `PauliEvolutionGate`s with no Trotter error of its own — the only Trotter
    error is from the 9 layers not commuting with each other. At half filling (`μ=U/2`), the
    interaction term's single-qubit `Z` pieces cancel exactly against the chemical-potential
    term, leaving a pure `(U/4)·Z_↑Z_↓` rotation per site (`append_interaction_gate`) plus a
    dropped global phase — both identities are checked numerically against `apply_hop`'s
    fermionic sign convention in the section's own derivation, not just asserted.
  - **Quench protocol: doublon decay, not a `h/J`-style field quench.** There's no swept field
    analogous to TFIM's `h`, so the quench is an interaction quench: start in a charge-density-
    wave Fock state (leg `y=0` fully doubly occupied, `y=1` empty — same half-filling sector as
    Section A) and evolve under fixed `U/t ∈ {1, 4, 8}`, watching double occupancy relax and
    charge redistribute across rungs (`doublon_decay_scan` for the circuit, `doublon_decay_exact`
    for the sector-restricted ED reference via `expm_multiply` — both conserve `(N_↑,N_↓)`
    exactly, so the comparison never leaves the 400-dim half-filling sector built in Section A).
    This is the standard "doublon decay" setup from cold-atom Hubbard experiments (e.g.
    Strohmaier et al. 2010); larger `U/t` visibly suppresses the decay — the dynamical
    counterpart to Section A's crossover, not a new physical claim. The leg-occupation plot's
    "balanced" reference line is `(N_UP_HALF+N_DOWN_HALF)/2`, computed dynamically, not
    hardcoded — if you change the filling, it still draws in the right place.
  - `PauliEvolutionGate` is opaque to Aer — every circuit is `.decompose(["PauliEvolution"])`'d
    before `sim.run(...)`, or the run raises `AerError: unknown instruction: PauliEvolution`.
- **Section C — Trotter error analysis**, implemented: `trotter_convergence_scan` fixes
  `U/t=4` (comparable hopping/interaction scales — the hardest case for the layers to not
  commute, the same reasoning as TFIM's choice of its critical `h/J=1`) and sweeps
  `N_STEPS_SCAN = [5, 10, 20, 40, 80, 160]`, measuring `|Trotter − exact|` double occupancy at
  `t=T_MAX`.
  - **Uses exact (shot-free) statevector evaluation, not Section B's shot-based measurement.**
    `double_occupancy_from_statevector` reads the circuit's final `Statevector` directly — at
    2000 shots, the statistical floor (order `1/sqrt(N_SHOTS*L)`) would mask the Trotter error
    exactly in the `Δt` range this analysis needs to resolve. This is the deliberate reason
    Section C doesn't reuse Section B's `run_circuit`/`double_occupancy_from_counts`.
  - **Empirical result, reported as measured, not forced to a textbook slope.** The fitted
    log-log slope in the asymptotic regime (`n_steps ≥ 40`) comes out close to **2** (`1.96` on
    the last run), steeper than the `O(Δt)` *global* error naively expected for a first-order
    product formula (whose `O(Δt²)` *local* error should generically accumulate to `O(Δt)`).
    **This was independently reproduced at the original 4×2 lattice too** (slope `1.98` there,
    on a system with a different layer count (7, not 9) and a different Hilbert space entirely)
    — not a coincidence of one lattice size. The section's closing markdown discusses this
    honestly as a plausible observable/state-specific cancellation of the leading error term,
    without a closed-form proof — don't "fix" the fitted-slope labels in this cell to match
    `O(Δt)` if re-running changes the numbers slightly; re-verify and update the prose instead.
  - **No second-order (symmetric Suzuki) scheme is implemented.** A prototype palindromic
    construction (generalizing TFIM's `X/2, ZZ, X/2` pattern to this Hamiltonian's many layers)
    was tried and showed a consistently *larger* absolute error than first order across the same
    `Δt` range — plausibly a large error-constant prefactor rather than a wrong asymptotic
    order, but not resolved in the time available, so it was left out rather than shipped
    unexplained. Picking this up again means budgeting time to either find the construction's
    actual bug or characterize the crossover `Δt` properly, not just reusing the prototype.
- **Section D — VQE**, implemented: a **Hamiltonian Variational Ansatz (HVA)** — the same
  hopping (`XX_str+YY_str`) and interaction (`ZZ`) gate primitives as Section B/C's Trotter
  circuit (`append_hop_gate`/`append_interaction_gate`, reused directly, not duplicated), but
  each gate's angle is an independent free parameter instead of a physically-fixed Trotter
  angle, applied to the same CDW reference state (`INIT_QUBITS`, Section B's) — rather than a
  generic hardware-efficient ansatz (matching theory doc §3's ask). This conserves `N_↑`/`N_↓`
  exactly for *any* parameter values (hopping gates conserve particle number per spin; the
  interaction gate is diagonal), hence also `Sz` exactly (not necessarily `S²`).
  - **`hubbard_hamiltonian_sparse_pauli`** builds the full Hamiltonian as a `SparsePauliOp`
    (summing the same per-bond/per-site terms Section B's circuit applies as gates) purely for
    VQE energy evaluation — ED itself stays in Section A's many-body occupation basis; these are
    two independent representations of the same physical `H`, cross-checked by construction
    (same `jw_hop_pauli_terms` calls), not by a runtime assertion.
  - **`N_PARAMS_PER_LAYER_VQE = 2·len(BONDS)+L`** (one angle per hopping bond, both spins, plus
    one per site's interaction term) **× `N_LAYERS_VQE=2`** layers = 48 parameters. `run_vqe`
    does `n_restarts` (default 3) independent COBYLA runs from random starts and keeps the best
    — empirically necessary: a single COBYLA run from one random start was not reliable at this
    ansatz size (see the lattice-size rationale above for the full investigation).
  - **Energy evaluation uses `Statevector.expectation_value` directly, not
    `qiskit.primitives.Estimator`.** Both give the identical exact, noiseless result; the
    `Statevector` path was empirically much faster per call at this circuit size during
    development, so that's what shipped. Don't swap to `Estimator` expecting a speedup.
  - **Reported honestly, not polished to look converged**: VQE gets the right qualitative trend
    (energy ↓, double occupancy ↓ with `U/t`) but with a persistent gap — energy off by roughly
    4–12%, double occupancy off by more and not even consistently signed (overestimates at
    `U/t=2`, overshoots downward at `U/t=4,8`). The notebook's own closing markdown ("Reading
    the comparison") states the exact numbers from the last run and the legitimate next steps
    (parameter-shift gradients, more restarts) that weren't pursued, in the interest of bounded
    runtime.
- **Limitations**, implemented: carries over the TFIM notebook's caveats (finite size, Trotter
  error, shot noise, ideal-simulator-vs-hardware, VQE non-convergence, boundary conventions) in
  Fermi-Hubbard-specific form, plus what's unique to this notebook — the crossover-not-transition
  framing (theory doc §1.3), the 3×2-vs-originally-planned-4×2 lattice compromise and its real
  cost (9 Trotter layers, not 7), that Sections C/D deliberately use shot-free exact evaluation
  (only Section B simulates the finite-sampling regime), and 2D JW Z-string/CNOT overhead not
  present in TFIM's 1D chain (theory doc §2.2/§2.6). Keep this section in sync with reality if
  changing `N`, shot counts, or optimizer settings — same discipline as TFIM's.
