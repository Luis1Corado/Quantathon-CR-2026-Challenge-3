# TFIM: ED, Trotter/Selene y VQE — Quantathon (QWorld Hackathon) CR 2026, Challenge 3

Estudio del **modelo de Ising con campo transverso (TFIM)** — su transición de fase cuántica
en el estado fundamental y su dinámica de *quench* (evolución temporal) — comparando una
línea base clásica de diagonalización exacta (ED) contra dos enfoques cuánticos:
evolución temporal Trotterizada (circuito ejecutado en el emulador **Selene** de Quantinuum
vía `guppylang`) y **VQE** con un *Hardware-Efficient Ansatz* (HEA).

Todo el análisis está consolidado en un único notebook:

```
TFIM_ED_Trotter_VQE_merged.ipynb
```

No hay ningún otro script ni notebook que ejecutar — este archivo genera, de principio a
fin, todas las figuras y resultados del entregable.

## Convención física

```
H = -J * Σ_i Z_i Z_{i+1}  -  h * Σ_i X_i        (condiciones de frontera periódicas)
```

`J = 1.0` se mantiene fijo; `h/J` es el parámetro barrido. El punto crítico está en
`h/J = 1`.

## Contenido del notebook

1. **Teoría** — el Hamiltoniano del TFIM y los observables reportados (`⟨Z⟩`, `⟨X⟩`,
   `⟨Z_i Z_{i+1}⟩_nn`).
2. **Sección A — Diagonalización exacta (línea base clásica):** diagrama de fases y
   evolución temporal exacta como referencia.
3. **Sección B — Evolución Trotterizada (guppylang + Selene):** circuito con el tamaño de
   paso `Δt` documentado explícitamente, dinámica de *quench* para `h/J ∈ {0.5, 1.0, 2.0}`,
   comparada contra ED.
4. **Sección C — Análisis del error de Trotter:** convergencia del error en función de `Δt`
   en el punto crítico.
5. **Sección D — VQE con Hardware-Efficient Ansatz:** barrido de `h/J` comparado contra ED.
6. **Limitaciones honestas** — tamaño finito, error de Trotter, ruido de shots, simulador
   ideal vs. hardware real, no-convergencia garantizada de VQE, convención de frontera.

Las secciones B y D incluyen además un apéndice **comentado por defecto** para ejecución en
Quantinuum Nexus (cloud); no es necesario para reproducir los resultados del notebook y
requiere una cuenta de Nexus con login interactivo (`qnx.login()`) — ver la nota dentro del
notebook si se desea explorar esa vía.

## Requisitos

- Python 3.11+ (probado con 3.14).
- Un entorno virtual (recomendado, para no ensuciar el Python del sistema).

Todas las dependencias están listadas en `requirements.txt`:

- `numpy`, `scipy`, `matplotlib` — álgebra lineal, diagonalización dispersa/densa (ED) y
  gráficas.
- `guppylang`, `selene-sim`, `hugr` — construcción de los circuitos cuánticos (Trotter y HEA)
  y su ejecución en el emulador Selene (Quest).
- `jupyter`, `notebook`, `ipykernel` — para abrir y correr el notebook.

## Instalación

### Linux

1. Verificar que hay un Python 3.11+ disponible:

   ```bash
   python3 --version
   ```

2. Si falta el módulo `venv` o `pip`, instalarlos con el gestor de paquetes de la
   distribución:

   ```bash
   # Debian / Ubuntu
   sudo apt update && sudo apt install python3-venv python3-pip

   # Fedora
   sudo dnf install python3-virtualenv python3-pip

   # Arch / Manjaro
   sudo pacman -S python-pip
   ```

3. Crear el entorno virtual, activarlo e instalar las dependencias:

   ```bash
   python3 -m venv quantathon_env
   source quantathon_env/bin/activate
   pip install -r requirements.txt
   ```

   Todas las dependencias (incluyendo `guppylang` y `selene-sim`) se instalan desde ruedas
   (`.whl`) precompiladas para Linux `manylinux` — no se requiere un compilador de C/C++ ni
   paquetes `-dev` adicionales.

4. Para desactivar el entorno cuando se termine de trabajar: `deactivate`.

### macOS / Windows

```bash
python3 -m venv quantathon_env
source quantathon_env/bin/activate        # Windows: quantathon_env\Scripts\activate
pip install -r requirements.txt
```

## Cómo correr el notebook

```bash
source quantathon_env/bin/activate
jupyter notebook TFIM_ED_Trotter_VQE_merged.ipynb
```

(o `jupyter lab` si se prefiere esa interfaz). Una vez abierto, **Run All** ejecuta el
notebook de principio a fin y regenera todas las figuras en el directorio raíz del
repositorio:

- `tfim_phase_diagram.png`
- `tfim_quench_h0.5_trotter_vs_exact.png`, `tfim_quench_h1.0_trotter_vs_exact.png`,
  `tfim_quench_h2.0_trotter_vs_exact.png`
- `trotter_error_convergence.png`
- `tfim_vqe_vs_ed_N6.png`

También puede ejecutarse de forma no interactiva:

```bash
jupyter nbconvert --to notebook --execute --inplace TFIM_ED_Trotter_VQE_merged.ipynb
```

### Tiempo de ejecución esperado

La Sección A (ED, `N=6`) corre en segundos. Las Secciones B, C y D compilan y ejecutan
circuitos reales en el simulador Selene (2000 shots por circuito) y son las que dominan el
tiempo total:

- Sección B (barrido de *quench* para 3 valores de `h/J`): del orden de **10 minutos**.
- Sección C (barrido de convergencia de Trotter): un par de minutos.
- Sección D (VQE, optimización COBYLA sobre ~11 valores de `h/J`): la más costosa,
  potencialmente **decenas de minutos**, ya que cada paso del optimizador requiere volver a
  compilar y ejecutar el circuito.

Para una corrida rápida de verificación, se puede reducir temporalmente `N_SHOTS`,
`STEP_COUNTS`/`N_STEPS_SCAN` (Secciones B/C) o `maxiter` (Sección D, definido en la llamada a
`run_vqe`) antes de correr todo el notebook.

## Notas de reproducibilidad

- Todo el notebook usa `N=6` de forma consistente entre ED, Trotter y VQE para que las
  comparaciones sean directas.
- El paso de Trotter `Δt = T_MAX / N_STEPS` está fijado explícitamente en la Sección B y se
  reutiliza (con su propio barrido) en el análisis de error de la Sección C.
- Los resultados de VQE dependen de una semilla fija (`seed=0`) para la inicialización de
  parámetros, pero al ser COBYLA un optimizador local, no hay garantía de mínimo global —
  ver la sección de limitaciones dentro del notebook.
