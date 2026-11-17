# Notas de estudio — Fermi-Hubbard en la escalera 3×2

Preguntas de repaso sobre conceptos del proyecto, con respuesta. Complementan el guion del póster
(`guion_poster_fermi_hubbard.md`); los números coinciden con el póster y el notebook.

---

## 1 · ¿Cómo funciona el mapeo de Jordan-Wigner?

**El problema.** Los fermiones **anticonmutan**: {c_i, c†_j} = δ_ij, y si dos fermiones
intercambian lugar la función de onda cambia de signo. Los qubits, en cambio, **conmutan** entre
sí. Jordan-Wigner es la forma de representar fermiones con qubits sin perder ese signo.

**Idea básica:**

1. **Un modo fermiónico, un qubit.** Un modo es un par (sitio, espín): 6 sitios × 2 espines = 12
   qubits.
2. **Ocupación = estado del qubit.** |1⟩ significa ocupado, |0⟩ vacío.
3. **El signo con una cadena de Z.** Cada operador fermiónico lleva antes una cadena de Z sobre
   todos los qubits anteriores. Esa cadena cuenta cuántos modos ocupados hay "antes" y devuelve
   el signo (−1)^(número de ocupados), justo el que exige la anticonmutación.

### Fórmulas

Primero se fija un orden lineal de los modos: 0, 1, …, 11.

```
c†_j = (Z_0 Z_1 ··· Z_{j-1}) · (X_j − iY_j)/2     ← crea: |0⟩→|1⟩ en el qubit j
c_j  = (Z_0 Z_1 ··· Z_{j-1}) · (X_j + iY_j)/2     ← destruye: |1⟩→|0⟩
```

**Operador número.** En n_j = c†_j c_j la cadena aparece dos veces y Z² = I, así que se cancela:

```
n_j = (I − Z_j)/2          ← siempre local, sin cadena
```

**Salto entre los modos i < j.** Solo sobreviven las Z **estrictamente entre** i y j:

```
c†_i c_j + h.c. = ½ [ X_i (Z_{i+1}···Z_{j-1}) X_j  +  Y_i (Z_{i+1}···Z_{j-1}) Y_j ]
```

Una compuerta tipo XX+YY "vestida" con una cadena de Z. Si i y j son vecinos en el orden, no
hay cadena.

### En el proyecto (escalera 3×2)

**Orden elegido:** primero todos los modos ↑ y luego todos los ↓; dentro de cada bloque, fila
por fila.

```
qubits 0–5  (↑):  (0,0) (1,0) (2,0) (0,1) (1,1) (2,1)
qubits 6–11 (↓):  igual
```

**El costo de 2D.** En 1D los vecinos físicos son vecinos en el orden. En 2D ningún orden lineal
logra eso para todos los enlaces. Por bloque de espín:

| Enlace | Ejemplo (qubits) | Cadena de Z |
|---|---|---|
| Pierna interior | 0–1, 1–2 | 0 |
| Pierna, enlace periódico | 2–0 | 1 (qubit 1) |
| Peldaño | 0–3 | 2 (qubits 1, 2) |

En total, **18 términos de salto**: 8 sin cadena, 4 con una Z y 6 con dos Z.

**La interacción no lleva cadena.** U n_{i↑} n_{i↓} es un producto de dos operadores número
locales: (1 − Z_↑ − Z_↓ + Z_↑Z_↓)/4, aunque los dos qubits estén a 6 posiciones de distancia.
Con μ = U/2, los términos de un qubit se cancelan y queda solo Z↑Z↓.

**Por qué ED y los circuitos coinciden.** En la Sección A, `apply_hop` calcula el signo de
c†_p c_q como la paridad de los modos ocupados con índice menor: exactamente la cadena
Π_{k<j} Z_k, con el mismo orden de qubits. Si se cambiara el orden en un solo lugar, los signos
dejarían de coincidir.

> **Para decirlo:** "Jordan-Wigner asigna un qubit a cada modo y reproduce el signo fermiónico
> con cadenas de Z. En 2D algunos enlaces quedan lejos en el orden y pagan cadenas más largas;
> en nuestra escalera, las más largas son de dos qubits, en los peldaños."

Derivación completa: `fermi_hubbard_theory.md` §2.1–2.3.

---

## 2 · ¿Qué significan la brecha de carga y la brecha de espín?

Las dos miden **cuánta energía cuesta excitar el estado base**, cada una para un tipo de
excitación distinto.

### Brecha de carga Δ_c

La energía que cuesta **agregar o quitar un electrón** (mover carga):

```
Δ_c = E(N+1) + E(N−1) − 2E(N)
```

- **Δ_c = 0:** mover carga no cuesta nada → se comporta como **metal**.
- **Δ_c > 0:** mover carga cuesta energía → **aislante**.

Con interacción U, un electrón extra obliga a crear una doble ocupación, que cuesta del orden
de U; por eso a U grande Δ_c crece casi como U. Es la firma de un **aislante de Mott**: aislante
por la repulsión entre electrones, no por las bandas.

**En el póster:** Δ_c = 0, 0.51, 1.04, 1.68, 4.24 para U/t = 0, 2, 4, 5, 8. Crece suavemente,
sin salto → *crossover*, no transición.

### Brecha de espín Δ_s

La energía que cuesta **cambiar el espín total sin mover carga** (voltear un espín):

```
Δ_s = E(S_z = 1) − E(S_z = 0)
```

- **Δ_s = 0:** excitar el espín es gratis, o el estado base ya es magnético.
- **Δ_s > 0:** los espines forman un estado no magnético (singlete) y romperlo cuesta energía.

**En el póster:** Δ_s = 0 hasta U/t ≈ 4.3, luego 0.09 (U/t = 5) y 0.30 (U/t = 8). **No crece
suavemente desde cero**: por debajo de 4.3 el estado base es un triplete (ver pregunta 5).

**Por qué ambas valen 0 en U = 0:** los niveles de un electrón son {−3, −1, 0, 0, 2, 2}. Con 3
electrones por espín, el nivel 0 (doble) queda a medio llenar, y agregar un electrón o
reacomodar espines ahí no cuesta energía. Es un efecto del tamaño pequeño de la red.

> **Para decirlo:** "La brecha de carga dice cuánto cuesta mover un electrón; la de espín,
> cuánto cuesta voltear un espín. En un aislante de Mott la carga queda congelada (Δ_c grande),
> pero el espín puede seguir siendo barato. Por eso se miden por separado."

---

## 3 · ¿Cómo se miden (calculan) las brechas?

En el proyecto **no se miden en el circuito**: se **calculan con diagonalización exacta** en la
Sección A del notebook (celda A.5, `analyze_fermi_hubbard_2d`). Ni Trotter ni VQE las calculan.

### En el notebook

Las brechas son **diferencias entre estados base de distintos sectores** (subespacios con N↑ y
N↓ fijos). Como H conserva N↑ y N↓, cada sector se diagonaliza por separado, sin tocar los
2¹² = 4096 estados completos.

**Paso 1. Construir cuatro sectores una sola vez** (su estructura no depende de U):

| Sector | (N↑, N↓) | Dimensión | Para qué |
|---|---|---|---|
| `HALF` | (3, 3) | 20 × 20 = 400 | estado base, E(N) |
| `NP1` | (4, 3) | 15 × 20 = 300 | E(N+1) |
| `NM1` | (2, 3) | 15 × 20 = 300 | E(N−1) |
| `SZ1` | (4, 2) | 15 × 15 = 225 | E(S_z = 1) |

**Paso 2.** Para cada U/t, armar H = −t·hop + U·n_int − μ·n_tot (μ = U/2) y buscar la energía
más baja de cada sector con `eigsh(H, k=1, which="SA")` (tipo Lanczos).

**Paso 3. Restar:**

```python
d_c = e_plus + e_minus - 2 * e0      # Δ_c = E(N+1) + E(N−1) − 2E(N)
d_s = e_sz1 - e0                     # Δ_s = E(Sz=1) − E(Sz=0)
```

Las 17 U/t juntas toman menos de un segundo.

- N±1 se construyen agregando/quitando un electrón ↑; por simetría ↑↔↓ daría lo mismo con ↓.
- **Δ_s = 0 no es un error numérico:** debajo de U/t ≈ 4.3 el triplete tiene componentes con
  S_z = 0 y S_z = 1 de la misma energía, así que e_sz1 = e0 exactamente.

### Con métodos cuánticos (no lo hicimos)

- **VQE:** el ansatz HVA conserva N↑ y N↓, así que bastaría correrlo desde un estado de
  referencia en otro sector y restar energías. Pero el error de nuestro VQE (4–12 % en la
  energía) sería **del mismo tamaño o mayor que las brechas**.
- **Evolución temporal (Trotter):** las energías aparecen como frecuencias de oscilación; se
  extraen con Fourier o estimación de fase. Requiere tiempos de evolución largos.

### En experimentos reales

Materiales: brecha de carga con **espectroscopía óptica o fotoemisión**; de espín con
**dispersión de neutrones**. Átomos fríos: **espectroscopía de modulación** (se hace oscilar la
red óptica y se ve a qué frecuencia el sistema absorbe energía).

> **Para decirlo:** "Las calculamos con diagonalización exacta: buscamos la energía mínima en
> sectores con un electrón más o menos para la brecha de carga, y con un espín volteado para la
> de espín, y restamos. Cada sector tiene entre 225 y 400 estados, así que es instantáneo. Con
> VQE se podría hacer igual, pero su error en la energía sería mayor que las brechas mismas."

---

## 4 · ¿Por qué son 9 capas por paso de Trotter?

**9 = 4 capas de salto por espín × 2 espines + 1 capa de interacción.**

### La regla: dentro de una capa, todo debe conmutar

Si los términos de una capa conmutan entre sí, la capa se aplica exacta; el error de Trotter
solo aparece **entre capas**. Dos saltos conmutan si **no comparten ningún sitio** (0–1 y 1–2
comparten el sitio 1 → capas distintas). Es una propiedad de los fermiones que Jordan-Wigner
conserva: los peldaños conmutan entre sí aunque sus cadenas de Z se crucen en el circuito.

### Piernas: 3 capas por espín

```
fila y=1:   3 ── 4 ── 5 ── (de vuelta a 3)
fila y=0:   0 ── 1 ── 2 ── (de vuelta a 0)
```

Cada fila es un **anillo de 3 sitios** (un triángulo): **cualquier par de enlaces comparte un
vértice**, así que cada enlace necesita su propia capa. Sí se juntan los enlaces equivalentes de
las dos filas:

| Capa | Enlaces |
|---|---|
| 1 | 0–1 y 3–4 |
| 2 | 1–2 y 4–5 |
| 3 | 2–0 y 5–3 (los que cierran el anillo) |

### Peldaños: 1 capa por espín

0–3, 1–4 y 2–5 no comparten sitios → una sola capa.

### Total

```
por espín:   3 (piernas) + 1 (peldaños) = 4 capas
× 2 espines                             = 8 capas
+ interacción U (todas las Z↑Z↓)        = 1 capa
                                        ─────────
                                          9 capas
```

Los términos n↑n↓ son diagonales y conmutan entre sí, pero no con los saltos: por eso una capa
aparte.

**Por qué en 4×2 eran 7.** Un anillo de 4 sitios sí es bipartito (2 colores): 0–1 y 2–3 en una
capa, 1–2 y 3–0 en otra. Son 2 + 1 = 3 por espín, × 2 = 6, + 1 = **7**. El anillo impar de 3
sitios obliga a una capa más por espín.

**Matiz para preguntas.** Los saltos ↑ y ↓ actúan sobre qubits distintos, así que **también
conmutan entre sí** (por eso en el dibujo del circuito aparecen en paralelo). Separarlos no
agrega error. Contando solo los grupos que de verdad no conmutan son 5 (3 de pierna, 1 de
peldaño, 1 de interacción). El "9" es la convención del notebook, la misma con la que salen los
7 de la red 4×2.

> **Para decirlo:** "Dentro de una capa todos los saltos deben conmutar, es decir, no compartir
> sitios. Cada pierna es un anillo de 3 sitios, un triángulo, donde cualquier par de enlaces
> comparte un vértice: 3 capas. Los peldaños caben en una. Eso da 4 por espín, 8 con los dos
> espines, más una de interacción: 9."

---

## 5 · ¿Qué son el singlete y el triplete?

### Dos espines

Un electrón tiene espín ½ (↑ o ↓). Dos electrones combinan sus espines en un espín total S,
que solo puede valer 0 o 1.

```
                  TRIPLETE (S = 1)                         SINGLETE (S = 0)
         "los espines suman, apuntan juntos"          "los espines se cancelan"

  m = +1:     ↑ ↑                                          ↑↓ − ↓↑
  m =  0:   (↑↓ + ↓↑)/√2                                   ───────
  m = −1:     ↓ ↓                                            √2

        3 estados, todos con la                       1 solo estado
        misma energía (degenerados)                   espín total cero, no magnético
```

- **Triplete:** espín total 1, como un pequeño imán; **3 versiones** según hacia dónde apunta
  (m = +1, 0, −1), todas con la misma energía sin campo magnético.
- **Singlete:** espín total 0, sin magnetismo; **una sola versión**, con los espines
  perfectamente antialineados.

**Lo confuso:** el estado m = 0 del triplete *también* tiene un ↑ y un ↓. Lo que lo distingue
del singlete es el **signo** de la superposición (+ en el triplete, − en el singlete). Contar ↑ y
↓ no basta.

Con 6 electrones aplica la misma idea: el estado base completo tiene S = 1 (triplete) o S = 0
(singlete) según el valor de U.

### U pequeño → triplete

```
 energía
   2   ─────   ─────          (vacíos)

   0   ──↑──   ──↑──    ← nivel DOBLE, con solo 2 electrones: medio lleno
                          los 2 electrones prefieren espines paralelos → TRIPLETE
  −1   ──↑↓──                 (lleno)
  −3   ──↑↓──                 (lleno)
```

Los niveles −3 y −1 se llenan con un par ↑↓ cada uno; quedan **2 electrones** para el nivel 0,
que es doble. Como en la **regla de Hund** de los átomos, con U pequeño prefieren espines
**paralelos**: Pauli los mantiene separados y se repelen menos. Es un **efecto de tamaño
finito** (el nivel 0 queda a medio llenar solo porque hay 6 sitios).

### U grande → singletes en los peldaños

Con U grande hay un electrón por sitio, pero les conviene "asomarse" virtualmente al vecino
para bajar su energía. **Solo pueden si los espines son opuestos** (Pauli prohíbe que dos ↑
compartan sitio, aunque sea un instante):

```
  espines antiparalelos:   ↑ ↓   →  ↑↓ _  →  ↑ ↓     ✓ puede asomarse, baja la energía
  espines paralelos:       ↑ ↑   →  ✗ (Pauli lo prohíbe)
```

Esto es el **superintercambio**, una atracción efectiva hacia espines opuestos con intensidad
J ≈ 4t²/U. Cada peldaño forma su propio singlete:

```
   y=1:   ↑   ↓   ↑            cada peldaño (línea vertical)
          |   |   |            es un par en SINGLETE
   y=0:   ↓   ↑   ↓            → espín total S = 0
```

### El cruce en U/t ≈ 4.3

```
 energía
   │  triplete ──────────╲
   │                      ╲  ╱── singlete
   │   singlete ───────────╳────            ← se cruzan: cruce de niveles
   │                      ╱  ╲── triplete
   └──────────────────────┼──────────→  U/t
                         4.3
     estado base: TRIPLETE │ SINGLETE
```

| Lo que se ve en el póster | Por qué |
|---|---|
| **Δ_s = 0** hasta 4.3 | El triplete ya tiene versiones m = 0 y m = 1 con la misma energía: voltear un espín es gratis. |
| **Δ_s > 0** después (0.09, 0.30) | Desde el singlete hay que romper un par para subir el espín. |
| **Correlación de peldaño** salta a −0.15 / −0.18 | Espines antialineados en los peldaños (singletes); negativo = opuestos. |

> **Para decirlo:** "Dos espines pueden sumar su espín, que es el triplete, o cancelarlo, que es
> el singlete. Con interacción débil, nuestra red tiene un nivel a medio llenar y los electrones
> se alinean, como en la regla de Hund: el estado base es un triplete. Con interacción fuerte
> prefieren espines opuestos con su vecino de peldaño y forman singletes. En U/t ≈ 4.3 se
> cruzan: ese es el salto que se ve en la gráfica."

---

## 6 · Si son 9 capas, ¿por qué el circuito tiene 18 cajas?

Una **capa** no es una compuerta: es un **grupo de compuertas que conmutan entre sí** y se
aplican juntas. Las 18 cajas del dibujo son los **18 términos de salto**, uno por enlace y por
espín (6 de pierna + 3 peldaños = 9 por espín), repartidos entre las 8 capas de salto:

| Capa | Enlaces | Compuertas |
|---|---|---|
| 1 (↑) | 0–1, 3–4 | 2 |
| 2 (↑) | 1–2, 4–5 | 2 |
| 3 (↑) | 2–0, 5–3 | 2 |
| 4 (↑) | peldaños 0–3, 1–4, 2–5 | 3 |
| 5–8 (↓) | lo mismo para espín ↓ | 2 + 2 + 2 + 3 |
| 9 | interacción Z↑Z↓, una por sitio | 6 (líneas rojas) |
| **Total** | | **18 saltos + 6 interacción = 24** |

**Por qué el dibujo no muestra 9 columnas.** El dibujador de Qiskit no conoce nuestras capas:
coloca cada compuerta en la primera columna donde haya espacio.

- **Las capas ↑ y ↓ quedan lado a lado**, porque usan qubits distintos.
- **Los 3 peldaños (una sola capa) salen en 3 columnas**: sus cajas se traslapan en los qubits
  de la cadena de Z (0–3, 1–4, 2–5). Conmutan igual, pero no se pueden dibujar una encima de otra.
- **Las 6 interacciones salen en 6 columnas** por la misma razón: cada línea roja cruza los
  qubits que hay entre i↑ e i↓.

**En el circuito real hay aún más compuertas.** El notebook aplica cada salto como dos
rotaciones (XX y YY con su cadena de Z): 36 + 6 = 42 compuertas de alto nivel, que compiladas a
compuertas básicas dan los **148 CNOT por paso** del póster.

> **Para decirlo:** "Una capa es un grupo de compuertas que conmutan y se aplican juntas. Las 9
> capas contienen 18 saltos, uno por enlace y espín, más 6 interacciones, una por sitio."
