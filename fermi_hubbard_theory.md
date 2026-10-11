# The Fermi-Hubbard Model: Theory and Fermion-to-Qubit Mapping

This document lays the theoretical groundwork for a new, separate notebook analyzing the
quantum phase transition and quench dynamics of the **2D Fermi-Hubbard model on a 3×2 lattice**
(originally planned at 4×2; reduced to 3×2 once Section D's VQE forced the issue — see
`CLAUDE.md` and the notebook's own intro/Section D for the empirical account), following the
same ED-vs-Trotter-vs-VQE structure used for the TFIM in
`TFIM_ED_Trotter_VQE_merged.ipynb`, but with a different toolchain: the Hamiltonian, the
fermion-to-qubit mapping, and the Trotter decomposition are **hand-derived** (as they were for
TFIM/guppylang), while **Qiskit** supplies the circuit primitives, simulator execution, and the
VQE classical-optimizer loop — no `guppylang`/Selene, no built-in `qiskit-nature` mappers or
Hamiltonian builders.

## 1. The model

### 1.1 General lattice Hamiltonian

For fermions of spin `σ ∈ {↑, ↓}` on any lattice of sites connected by a bond set `⟨i,j⟩`:

```
H = -t * Σ_{⟨i,j⟩,σ} (c†_{i,σ} c_{j,σ} + c†_{j,σ} c_{i,σ})
    + U * Σ_i n_{i,↑} n_{i,↓}
    - μ * Σ_{i,σ} n_{i,σ}
```

- `c_{i,σ}`, `c†_{i,σ}` obey canonical fermionic anticommutation relations
  `{c_{i,σ}, c†_{j,σ'}} = δ_{ij} δ_{σσ'}`, `{c_{i,σ}, c_{j,σ'}} = 0`.
- `n_{i,σ} = c†_{i,σ} c_{i,σ}`.
- `t`: hopping amplitude (kinetic scale) — the analogue of TFIM's `J`.
- `U`: on-site Coulomb repulsion (interaction scale) — the analogue of TFIM's `h`. `U/t` is the
  swept coupling.
- `μ`: chemical potential; `μ = U/2` gives half filling (one fermion/site on average) by
  particle-hole symmetry.

This is dimension-agnostic — a 1D chain is the special case where `⟨i,j⟩` is a path (or ring,
with PBC); the 2D case below just uses a richer bond set.

### 1.2 This project's lattice: a 3×2 Hubbard ladder

Sites are indexed `(x, y)` with `x = 0..2` (**periodic**, the "leg"/ring direction) and
`y = 0, 1` (**open**, the "rung" direction — 2 sites can't support a genuine second periodic
bond without doubling up on the same pair, so this direction stays open). This is the standard
**2-leg Hubbard ladder with periodic legs**, 6 sites total:

```
(0,0)-(1,0)-(2,0)-(0,0)   [leg y=0, ring of 3]
  |     |     |
(0,1)-(1,1)-(2,1)-(0,1)   [leg y=1, ring of 3]
```

Bond inventory (9 nearest-neighbor bonds total):

- **Leg bonds** (within a row): `(x,y)-(x+1 mod 3, y)` for `x=0..2`, `y=0,1` → 3 bonds/leg × 2
  legs = **6 leg bonds**.
- **Rung bonds** (between rows): `(x,0)-(x,1)` for `x=0..2` → **3 rung bonds**.

With 2 spin species, that's `9 × 2 = 18` hopping terms, plus `6` on-site interaction terms
(one per site) and half filling means `N_↑ = N_↓ = 3` fermions (`N = 6` total, `N = L`).

Why 3×2 rather than a larger ladder: ED and Trotter alone would have been comfortable at 4×2
(8 sites, 16 qubits) — it's Section D's VQE that forced the reduction. A Hamiltonian
Variational Ansatz at 4×2, with COBYLA (matching TFIM's optimizer choice), did not converge in
affordable time even at the easy `U=0` free-fermion check; a smaller, cheaper-per-circuit
lattice made a more expressive ansatz (one free angle per bond) and multiple optimizer restarts
affordable. See the notebook's Section D intro and `CLAUDE.md` for the full empirical account —
not repeated here since this file is about the physics/mapping, not the optimizer. One
structural consequence of the smaller ring *does* belong here: see §2.4.

### 1.3 Physical picture: crossover, not a sharp transition

**Honest caveat, carried over and sharpened from the 1D case:** this is a 2-leg ladder, not a
true extended 2D lattice, and 2-leg Hubbard/Heisenberg ladders are well known to be **gapped for
any `U > 0`** — at large `U/t` the effective model is a two-leg Heisenberg ladder, whose ground
state is a **rung-singlet state with a finite spin gap**, adiabatically connected all the way
down to `U → 0⁺`. So, exactly as in the 1D chain, there is **no finite-`U_c` phase transition**
to look for at half filling here either — the honest framing is again a **crossover**, now
diagnosable through ladder-specific observables:

- **Double occupancy** `⟨n_↑ n_↓⟩` (as in 1D): decreases smoothly from `1/4` toward `0` as
  `U/t` grows.
- **Finite-size charge gap** `Δ_c(L) = E₀(N+1) + E₀(N-1) − 2E₀(N)` (as in 1D): positive for any
  `U > 0`.
- **Spin gap** `Δ_s = E₀(S_tot=1) − E₀(S_tot=0)`: the hallmark ladder diagnostic — the energy
  cost of promoting the singlet ground state to the lowest triplet. Grows with `U/t` as rung
  singlets strengthen; a genuinely 2D-physics observable that the 1D chain didn't have.
- **Rung singlet correlation** `⟨S_{(x,0)} · S_{(x,1)}⟩`: becomes more strongly (negatively)
  correlated as `U/t` increases, the real-space signature of rung-singlet formation. Preferred
  over a momentum-space structure factor like `S(π,π)` here — that quantity presumes true
  long-range 2D order, which a finite 2-leg ladder (quasi-1D, short-range correlated) does not
  have; reporting it would overstate what this cluster can show.

If a genuine finite-coupling transition is wanted later, the same 1D-doc caveats apply: an
**ionic** (staggered on-site potential) or **extended** (nearest-neighbor `V`) Hubbard variant
would introduce one. Not pursued here.

### 1.4 Observables to compute

- Site/spin occupation `⟨n_{i,σ}⟩`.
- Double occupancy `⟨n_{i,↑} n_{i,↓}⟩`, averaged over sites.
- Charge gap `Δ_c(L)` and spin gap `Δ_s(L)` (both require ED in adjacent particle-number/spin
  sectors, see §3).
- Rung correlation `⟨S_{(x,0)} · S_{(x,1)}⟩` and leg correlation `⟨S_{(x,y)} · S_{(x+1,y)}⟩`.
- Kinetic energy per bond `⟨c†_{i,σ} c_{j,σ} + h.c.⟩`.

## 2. Fermion-to-qubit mapping: Jordan-Wigner

### 2.1 Single-mode formulas (unchanged from the 1D case)

Fix a linear ordering `1, …, M` of the `M = 2×6 = 12` fermionic modes (one per site-spin pair).
For mode `j`:

```
c_j  = (Π_{k<j} Z_k) · (X_j + i Y_j)/2
c†_j = (Π_{k<j} Z_k) · (X_j − i Y_j)/2
n_j  = c†_j c_j = (I − Z_j)/2        (always local — the string self-cancels)
```

For two modes `i < j`:

```
c†_i c_j + c†_j c_i = (1/2) [ X_i (Z_{i+1}···Z_{j-1}) X_j + Y_i (Z_{i+1}···Z_{j-1}) Y_j ]
```

— an `XX+YY` "hopping gate" dressed with a `Z`-string over every mode strictly between `i` and
`j` in the ordering.

### 2.2 The core 2D problem: no ordering makes every bond local

In 1D, the physical chain order *is* a valid JW order, so nearest-neighbor bonds are always
adjacent in the mode ordering (string-free, under the blocked spin ordering from that case). In
2D there is no such luck: **any linear ordering of a 2D lattice's sites will place some
physically-adjacent sites far apart along the line**, and those bonds pick up nontrivial
`Z`-strings. This is an unavoidable, dimension-driven cost that the 1D chain didn't have — worth
stating plainly rather than glossing over.

**Chosen ordering** (spin-blocked, then row-major within each block):

```
↑ block (positions 0-5):   (0,0)↑ (1,0)↑ (2,0)↑ (0,1)↑ (1,1)↑ (2,1)↑
↓ block (positions 6-11):  (0,0)↓ (1,0)↓ (2,0)↓ (0,1)↓ (1,1)↓ (2,1)↓
```

Spin-blocking keeps the on-site interaction term string-free algebraically (per §2.1, a product
of two local number operators — see the 1D doc's §2.2(a) reasoning, unchanged by dimension),
even though the two qubits involved (`i` in the `↑` block, `i` in the `↓` block) are 6 apart in
the register. Row-major ordering within each block is the simplest 2D flattening.

### 2.3 Bond-by-bond string cost, worked out for this lattice

Per spin block (positions 0-5, pattern repeats identically for 6-11):

| Bond type | Example | Positions | String length |
|---|---|---|---|
| Leg, interior | `(0,y)-(1,y)`, `(1,y)-(2,y)` | adjacent | **0** |
| Leg, wraparound | `(2,y)-(0,y)` | 2 apart (e.g. 2,0) | **1** |
| Rung | `(x,0)-(x,1)` | 3 apart (e.g. 0,3) | **2** |

So per spin: 4 leg bonds are string-free, 2 leg bonds (the PBC wraparounds, one per row) need a
length-1 string, and all 3 rung bonds need a length-2 string. Doubled for both spins: **18
hopping terms**, of which 8 are string-free, 4 carry a 1-`Z` string, and 6 carry a 2-`Z` string.
Shorter strings than the 4×2 case across the board (a direct consequence of the smaller ring),
but still the same unavoidable 2D overhead flagged in §2.2 — no bond here is as cheap as every
1D bond was.

### 2.4 Trotter layer structure

Group terms into mutually-commuting layers (bonds sharing a qubit don't commute). This is where
the smaller lattice costs something back: a 4-site ring is **bipartite** (its 4 edges 2-color
into disjoint "even"/"odd" matchings), but a **3-site ring is not** — with only 3 vertices, any
2 of its 3 edges share one, so no 2 of a row's leg bonds can ever share a layer. The best
grouping is one layer per edge *position*: row `y=0`'s edge `k` and row `y=1`'s edge `k` touch
entirely disjoint qubits (different rows), so those two share a layer; the 3 positions
themselves cannot be merged further.

- **Leg layers** (per spin, 3 of them): `{(0,y)-(1,y) for y=0,1}`, `{(1,y)-(2,y) for y=0,1}`,
  `{(2,y)-(0,y) for y=0,1}` — 2 disjoint bonds each, 1 layer each.
- **Rung** (per spin): all 3 rung bonds are mutually disjoint (distinct `x`) — 1 layer.
- **Interaction**: all 6 on-site terms are diagonal → 1 layer.

That's **4 hopping sublayers per spin × 2 spins + 1 interaction layer = 9 non-commuting Trotter
layers** total — *more* than the 4×2 lattice's 7 (theory doc's earlier draft), the direct price
of the 3-site ring's non-bipartiteness. Still far short of the naive "one layer per bond" upper
bound (18+6=24) that ignoring all disjointness would give. This is the layer count implemented
in the Qiskit circuit.

### 2.5 Qubit count and simulation cost

`2 × 6 = 12` qubits → a `2^12 = 4096`-dimensional Hilbert space. Lighter than the original 4×2
plan's `2^16 = 65536` (and the fixed-sector ED basis is smaller too: `C(6,3)^2 = 400` at half
filling vs. `C(8,4)^2 = 4900`) — the direct payoff of the size reduction Section D's VQE forced.
Still meaningfully heavier than TFIM's `N=6` (`64`-dim) for the *circuit* sections (Trotter
quench sweeps, and especially VQE's many evaluations per optimizer step); budget shot counts /
optimizer `maxiter` accordingly (mirrors the runtime caveats already in the TFIM README).

### 2.6 Other approaches (context only, not implemented)

- **Parity / Bravyi-Kitaev mappings**: shorten average string length to `O(log M)`; not used
  here to keep the mapping transparent and hand-derivable.
- **Fermionic SWAP networks** (Kivlichan et al.): the standard way to get linear-depth 2D
  fermionic simulation by dynamically re-sorting qubit-to-mode assignment with SWAP gates so
  every needed bond becomes momentarily adjacent. This is the "real" fix to §2.2's problem for
  larger lattices; out of scope for a 6-site cluster but worth citing in the Limitations
  section as the path to scaling this up.

## 3. What this feeds into

- **ED baseline**: hand-built many-body Hamiltonian (fermionic occupation basis restricted to
  fixed `(N_↑, N_↓)`, or the JW-mapped Pauli sum as a cross-check — both in plain numpy/scipy, no
  Qiskit). Sparse diagonalization (`scipy.sparse.linalg.eigsh`) gives `E₀(N_↑,N_↓)` for the
  sectors needed for `Δ_c` and, via the `S_tot=0` vs `S_tot=1` sectors, `Δ_s`.
- **Trotter circuit**: the 9 layers of §2.4, each built as explicit `XX+YY` (hopping) or
  `ZZ`-type (interaction) rotations dressed with the string lengths from §2.3, assembled as a
  Qiskit `QuantumCircuit` from primitive gates (or `PauliEvolutionGate` used purely as a
  building block for a hand-specified Pauli term — not an auto-mapper). Executed via Qiskit Aer.
- **VQE ansatz**: should conserve total particle number and, ideally, total spin — a
  hopping-gate/Givens-rotation-based ansatz built from the same JW mapping is preferred over a
  generic hardware-efficient ansatz, which would explore unphysical number/spin sectors unless
  explicitly constrained. Implemented as a Hamiltonian Variational Ansatz: the same hopping/
  interaction gates as the Trotter circuit, with each angle promoted to a free parameter.
  Energy expectation values are computed exactly via `qiskit.quantum_info.Statevector`
  (`expectation_value`) rather than the `primitives.Estimator` wrapper — empirically much
  cheaper per call at this circuit size, with an identical (exact, noiseless) result; the
  classical optimizer loop (COBYLA, scipy) is unchanged in spirit from the plan.
