# The Fermi-Hubbard Model: Theory and Fermion-to-Qubit Mapping

This document lays the theoretical groundwork for a new, separate notebook analyzing the
quantum phase transition and quench dynamics of the **2D Fermi-Hubbard model on a 4×2 lattice**,
following the same ED-vs-Trotter-vs-VQE structure used for the TFIM in
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

### 1.2 This project's lattice: a 4×2 Hubbard ladder

Sites are indexed `(x, y)` with `x = 0..3` (**periodic**, the "leg"/ring direction) and
`y = 0, 1` (**open**, the "rung" direction — 2 sites can't support a genuine second periodic
bond without doubling up on the same pair, so this direction stays open). This is the standard
**2-leg Hubbard ladder with periodic legs**, 8 sites total:

```
(0,0)-(1,0)-(2,0)-(3,0)-(0,0)   [leg y=0, ring of 4]
  |     |     |     |
(0,1)-(1,1)-(2,1)-(3,1)-(0,1)   [leg y=1, ring of 4]
```

Bond inventory (12 nearest-neighbor bonds total):

- **Leg bonds** (within a row): `(x,y)-(x+1 mod 4, y)` for `x=0..3`, `y=0,1` → 4 bonds/leg × 2
  legs = **8 leg bonds**.
- **Rung bonds** (between rows): `(x,0)-(x,1)` for `x=0..3` → **4 rung bonds**.

With 2 spin species, that's `12 × 2 = 24` hopping terms, plus `8` on-site interaction terms
(one per site) and half filling means `N_↑ = N_↓ = 4` fermions (`N = 8` total, `N = L`).

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

Fix a linear ordering `1, …, M` of the `M = 2×8 = 16` fermionic modes (one per site-spin pair).
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
↑ block (positions 0-7):  (0,0)↑ (1,0)↑ (2,0)↑ (3,0)↑ (0,1)↑ (1,1)↑ (2,1)↑ (3,1)↑
↓ block (positions 8-15): (0,0)↓ (1,0)↓ (2,0)↓ (3,0)↓ (0,1)↓ (1,1)↓ (2,1)↓ (3,1)↓
```

Spin-blocking keeps the on-site interaction term string-free algebraically (per §2.1, a product
of two local number operators — see the 1D doc's §2.2(a) reasoning, unchanged by dimension),
even though the two qubits involved (`i` in the `↑` block, `i` in the `↓` block) are 8 apart in
the register. Row-major ordering within each block is the simplest 2D flattening.

### 2.3 Bond-by-bond string cost, worked out for this lattice

Per spin block (positions 0-7, pattern repeats identically for 8-15):

| Bond type | Example | Positions | String length |
|---|---|---|---|
| Leg, interior | `(0,y)-(1,y)`, `(1,y)-(2,y)` | adjacent | **0** |
| Leg, wraparound | `(3,y)-(0,y)` | 3 apart (e.g. 3,0) | **2** |
| Rung | `(x,0)-(x,1)` | 4 apart (e.g. 0,4) | **3** |

So per spin: 6 leg bonds are string-free, 2 leg bonds (the PBC wraparounds) need a length-2
string, and all 4 rung bonds need a length-3 string. Doubled for both spins: **24 hopping
terms**, of which 12 are string-free, 4 carry a 2-`Z` string, and 8 carry a 3-`Z` string. This
is the concrete, unavoidable 2D overhead flagged in §2.2 — rung hopping gates cost more CNOTs
than any 1D bond did.

### 2.4 Trotter layer structure

Group terms into mutually-commuting layers (bonds sharing a qubit don't commute):

- **Leg-even** (per spin): `{(0,y)-(1,y), (2,y)-(3,y)}` for both `y` — 4 disjoint bonds, 1 layer.
- **Leg-odd** (per spin): `{(1,y)-(2,y), (3,y)-(0,y)}` for both `y` — 4 disjoint bonds, 1 layer.
- **Rung** (per spin): all 4 rung bonds are mutually disjoint (distinct `x`) — 1 layer.
- **Interaction**: all 8 on-site terms are diagonal → 1 layer.

That's **3 hopping sublayers per spin × 2 spins + 1 interaction layer = 7 non-commuting Trotter
layers** total (vs. TFIM's 2, and better than the naive "up to 9" upper bound from not
exploiting bond-disjointness). This is the layer count to implement in the Qiskit circuit.

### 2.5 Qubit count and simulation cost

`2 × 8 = 16` qubits → a `2^16 = 65536`-dimensional Hilbert space. This is meaningfully heavier
than TFIM's `N=6` (`64`-dim): ED sparse diagonalization is still routine, but VQE (many circuit
evaluations per optimizer step) and Trotter quench sweeps will be the dominant runtime cost —
budget accordingly when picking shot counts / optimizer `maxiter` (mirrors the runtime caveats
already in the TFIM README).

### 2.6 Other approaches (context only, not implemented)

- **Parity / Bravyi-Kitaev mappings**: shorten average string length to `O(log M)`; not used
  here to keep the mapping transparent and hand-derivable.
- **Fermionic SWAP networks** (Kivlichan et al.): the standard way to get linear-depth 2D
  fermionic simulation by dynamically re-sorting qubit-to-mode assignment with SWAP gates so
  every needed bond becomes momentarily adjacent. This is the "real" fix to §2.2's problem for
  larger lattices; out of scope for an 8-site cluster but worth citing in the Limitations
  section as the path to scaling this up.

## 3. What this feeds into

- **ED baseline**: hand-built many-body Hamiltonian (fermionic occupation basis restricted to
  fixed `(N_↑, N_↓)`, or the JW-mapped Pauli sum as a cross-check — both in plain numpy/scipy, no
  Qiskit). Sparse diagonalization (`scipy.sparse.linalg.eigsh`) gives `E₀(N_↑,N_↓)` for the
  sectors needed for `Δ_c` and, via the `S_tot=0` vs `S_tot=1` sectors, `Δ_s`.
- **Trotter circuit**: the 7 layers of §2.4, each built as explicit `XX+YY` (hopping) or
  `ZZ`-type (interaction) rotations dressed with the string lengths from §2.3, assembled as a
  Qiskit `QuantumCircuit` from primitive gates (or `PauliEvolutionGate` used purely as a
  building block for a hand-specified Pauli term — not an auto-mapper). Executed via Qiskit Aer.
- **VQE ansatz**: should conserve total particle number and, ideally, total spin — a
  hopping-gate/Givens-rotation-based ansatz built from the same JW mapping is preferred over a
  generic hardware-efficient ansatz, which would explore unphysical number/spin sectors unless
  explicitly constrained. Execution (parameter binding, expectation values via Qiskit's
  `Estimator`, classical optimizer loop) uses Qiskit; the ansatz structure itself is hand-designed.
