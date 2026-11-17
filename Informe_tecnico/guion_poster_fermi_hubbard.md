# Guion de la presentación — Póster Fermi-Hubbard (escalera 3×2)

Charla de **10 minutos** frente a `poster_fermi_hubbard.pdf`. El póster solo lleva figuras,
tablas y una frase clave por sección; **todo lo que hay que explicar está aquí**.

- Ritmo: ~140 palabras por minuto. Cada sección indica el tiempo y **a qué señalar** (👉).
- El texto entre comillas es para decirlo tal cual o casi. Los puntos sueltos son apoyo.
- Al final hay **preguntas probables con respuesta** y **datos de respaldo**.

---

## 0 · Apertura (0:00 – 0:30)

👉 Título y pregunta del encabezado.

> "Nuestro proyecto responde una pregunta concreta: **¿qué tan lejos llegan los métodos
> cuánticos frente a la solución exacta?** Tomamos un modelo de materiales fuertemente
> correlacionados, el modelo de Fermi-Hubbard, y lo resolvimos de tres formas: de forma exacta
> en una computadora clásica, con evolución temporal en circuitos cuánticos (Trotter) y con un
> algoritmo variacional (VQE). Después comparamos los tres."

---

## 1 · Modelo y métodos (0:30 – 2:00)

👉 La ecuación y el dibujo de la escalera.

> "El Hamiltoniano tiene dos términos que compiten. El primero, con *t*, deja que los
> electrones salten entre sitios vecinos: favorece que se muevan. El segundo, con *U*, castiga
> que dos electrones compartan un sitio: favorece que se queden quietos, uno por sitio. Toda la
> física sale de la competencia entre *t* y *U*, así que fijamos *t* = 1 y barremos *U/t*."

> "La red es una escalera de 3 por 2: 6 sitios, con las piernas periódicas y los peldaños
> abiertos. Cada sitio puede tener un electrón con espín arriba y otro con espín abajo, así que
> son 12 modos fermiónicos y, con la transformación de Jordan-Wigner, **12 qubits**. La
> transformación la derivamos a mano, no con una librería automática."

- Trabajamos a **llenado medio**: 3 electrones con espín arriba y 3 abajo, con μ = U/2.
- En 2D, Jordan-Wigner deja algunos saltos entre qubits no vecinos: 10 de los 18 términos de
  salto llevan una "cadena de Z" de 1 o 2 qubits, que cuesta compuertas extra.

👉 Tabla "plan 4×2 / usado 3×2".

> "El plan original era una escalera de 4×2, 16 qubits. Lo que nos obligó a reducirla fue el
> VQE: a 16 qubits no convergía ni en el caso más fácil, *U* = 0. Llegaba a −5.3 cuando la
> respuesta exacta es −12. A 12 qubits cada circuito cuesta unos 0.04 s en vez de 0.3–0.7 s, y
> eso nos permitió un ansatz más expresivo y varios reinicios del optimizador. Para comparar
> lo mismo con los tres métodos, cambiamos **todo** el proyecto a 3×2."

> "Esto tuvo un costo inesperado: un anillo de 3 sitios **no es bipartito**. Dos enlaces
> cualesquiera de un triángulo comparten un vértice, así que no se pueden aplicar a la vez. Por
> eso cada paso de Trotter necesita **9 capas** de compuertas en vez de 7. La red más chica no
> es más barata en todo, solo donde más importaba."

👉 Tabla de métodos.

> "ED nos da la respuesta exacta y sirve de referencia. Trotter responde preguntas de
> dinámica, y VQE busca el estado base."

---

## 2 · Diagonalización exacta: el crossover (2:00 – 3:30)

👉 Las tres gráficas, de izquierda a derecha.

> "Primero, la referencia exacta. A llenado medio el problema se reduce a un espacio de solo
> 400 estados, así que el barrido completo en *U/t* toma menos de un segundo."

> "A la izquierda, la **doble ocupación**: la probabilidad de encontrar dos electrones en el
> mismo sitio. Sin interacción vale 0.25, y al subir *U* cae hasta 0.04: los electrones se
> evitan. En el centro, la **brecha de carga** crece de 0 a 4.2, y a *U* grande crece casi
> como *U*. Esa es la firma de un aislante tipo Mott: mover carga cuesta energía."

> "Algo importante: en una escalera de dos piernas **no hay transición de fase**. No hay un
> *U* crítico donde algo cambie bruscamente; es un **crossover** suave. Por eso hablamos de
> 'crossover' y no de 'transición'."

👉 La línea punteada en *U/t* ≈ 4.3 y la fila "Estado base" de la tabla.

> "Pero sí aparece algo brusco en *U/t* ≈ 4.3: la correlación de espín en los peldaños salta,
> la doble ocupación hace un quiebre y la brecha de espín, que valía exactamente cero, se
> abre. Investigamos qué era: es un **cruce de niveles por tamaño finito**."

> "Los niveles de energía de un electrón en esta red son −3, −1, 0, 0, 2 y 2. Con 3 electrones
> por espín se llenan −3 y −1, pero el nivel 0, que es doble, queda **a medio llenar**. Los
> dos electrones de ese nivel prefieren alinear su espín, como en la regla de Hund. Por eso,
> hasta *U/t* ≈ 4.3, el estado base es un **triplete**. Por encima, gana un **singlete de
> peldaño**, que es la física esperada en una escalera grande. Lo verificamos comparando las
> energías de los sectores de espín 0, 1 y 2. No es un error del código, es un efecto real de
> usar solo 6 sitios."

---

## 3 · Evolución de Trotter: decaimiento de doblones (3:30 – 6:00) ⭐ sección principal

👉 La línea de parámetros sobre las gráficas.

> "Ahora, la parte cuántica que mejor funcionó. Hicimos un experimento de **quench**: empezamos
> con la pierna de abajo llena, dos electrones en cada sitio (lo que llamamos *doblones*), y la
> pierna de arriba vacía. La doble ocupación inicial es 0.5. Después dejamos que el sistema
> evolucione y vemos qué pasa con los doblones."

> "Es el mismo protocolo de los experimentos con átomos fríos en redes ópticas, por ejemplo
> el de Strohmaier y colaboradores en 2010, así que hay física real que comparar."

- Circuito construido a mano: cada salto es el producto de dos rotaciones (XX e YY con su
  cadena de Z). Como esas dos conmutan, la factorización de cada enlace es **exacta**. El único
  error de Trotter viene de que las 9 capas no conmutan entre sí.
- La interacción, a llenado medio, se reduce a una sola rotación Z↑Z↓ por sitio: los términos
  de un qubit se cancelan exactamente con el potencial químico.
- 50 pasos hasta *T* = 5, simulado en Qiskit Aer con 2000 mediciones (shots) por circuito.

👉 Gráfica izquierda (*U/t* = 1).

> "Con interacción débil, *U/t* = 1, los doblones **decaen**: la doble ocupación baja de 0.5 a
> unos 0.26 y oscila. En el panel derecho se ve la carga pasando de una pierna a la otra, como
> si el sistema 'respirara'."

👉 Gráfica derecha (*U/t* = 8).

> "Con interacción fuerte, *U/t* = 8, los doblones **casi no decaen**: la doble ocupación se
> queda entre 0.46 y 0.50. ¿Por qué? Romper un doblón libera una energía *U* grande, y el
> sistema no tiene dónde ponerla: los saltos solo dan energías del orden de *t*. Así que los
> doblones quedan atrapados. Fíjense que la carga sí se sigue moviendo un poco, pero los pares
> no se rompen."

👉 Puntos (circuito) sobre la línea continua (exacto).

> "Lo importante para nuestra pregunta: los puntos del circuito caen encima de la curva exacta.
> La diferencia típica es de 0.001 a 0.005, que es del tamaño del ruido estadístico de 2000
> shots. Es decir, **el circuito de Trotter reproduce la dinámica exacta**, y además captura un
> fenómeno físico real."

---

## 4 · El circuito de un paso de Trotter (6:00 – 7:15)

👉 El circuito, de izquierda a derecha. Los qubits 0↑–5↑ son los electrones con espín arriba y
0↓–5↓ los de espín abajo.

> "Así se ve **un paso de Trotter** en compuertas cuánticas, dibujado con Qiskit. Cada caja es
> un salto de un electrón entre dos sitios vecinos, y las líneas rojas de la derecha son la
> interacción *U*: una rotación Z↑Z↓ por sitio."

> "Lo importante es la **altura de las cajas**. Por Jordan-Wigner, cada salto lleva una cadena
> de Z sobre los qubits que quedan en medio. Los saltos de pierna (morado) tocan 2 qubits; los
> que cierran el anillo (turquesa), 3; y los de peldaño (azul oscuro), 4. Ese costo extra
> aparece por ser una red 2D: en una cadena 1D no existe."

> "Los saltos que no comparten qubits van en paralelo. Por eso los de espín arriba y abajo
> quedan uno al lado del otro. En total son **9 capas** por paso, y no 7 como en la red 4×2,
> porque un anillo de 3 sitios no se puede colorear con 2 colores."

> "Al compilarlo a compuertas básicas, un paso tiene **148 CNOT**. Los 50 pasos de la
> simulación suman unas **7400**. En un simulador ideal no es problema, pero en hardware actual,
> con ruido, sí lo sería. Correrlo en hardware real es el siguiente paso."

- Si preguntan por el error de Trotter: está en el notebook (Sección C). Con *U/t* = 4, el
  error baja de 0.026 (5 pasos) a 0.00025 (160 pasos). Ver la pregunta sobre la pendiente 2.

---

## 5 · VQE (7:15 – 8:45)

👉 Panel de energía, luego el de doble ocupación (las diferencias numéricas no están en el póster: dílas tú).

> "Por último, el VQE. Usamos un **ansatz Hamiltoniano-Variacional**: las mismas compuertas
> del circuito de Trotter, pero con cada ángulo libre. Son 48 parámetros. La ventaja es que
> conserva el número de electrones de cada espín por construcción, así que el optimizador nunca
> se sale del sector físico correcto."

> "El resultado: VQE sigue la **tendencia correcta**. La energía baja y la doble ocupación cae
> al subir *U*. Pero queda una **brecha persistente** de más o menos +1 en la energía, entre 4
> y 12 % en términos relativos. En la doble ocupación es peor: a *U/t* = 2 la sobreestima, y a
> *U/t* = 4 y 8 la subestima. Falla en ambas direcciones."

> "Comprobamos el Hamiltoniano y el circuito contra la solución exacta, así que el problema no
> está ahí, sino en el **optimizador**. COBYLA, aun con 3 reinicios aleatorios y 600
> iteraciones cada uno, no encuentra el mínimo global de este paisaje de energía. Cada valor de
> *U* ya cuesta más de un minuto de cómputo."

- Si preguntan por el punto exacto en *U* = 0 de la doble ocupación: ahí el estado base es
  degenerado, así que el valor (≈0.25) depende de qué estado del subespacio devuelva el
  algoritmo y cambia ligeramente de una corrida a otra.

---

## 6 · Conclusiones (8:45 – 10:00)

👉 El bloque de conclusiones, un punto a la vez.

> "Para cerrar, volvemos a la pregunta: ¿qué tan lejos llegan los métodos cuánticos?"

1. > "**Trotter funciona.** Reproduce la dinámica exacta dentro del ruido de medición y captura
   > física real: la supresión del decaimiento de doblones."
2. > "**El circuito lo construimos a mano**: Jordan-Wigner, 9 capas por paso y unos 148
   > CNOT, sin un mapeo automático."
3. > "**VQE es el cuello de botella.** Capta la tendencia, pero no la precisión, y su costo
   > decidió el tamaño de todo el proyecto."
4. > "**A 6 sitios domina el tamaño finito.** Hasta *U/t* ≈ 4.3 el estado base es un triplete,
   > y la física de escalera esperada solo aparece después."

> "Como siguientes pasos: gradientes con la regla de *parameter-shift* para el VQE, un esquema
> de Trotter de segundo orden, redes más grandes usando *fermionic SWAP networks* y, sobre
> todo, correr en hardware real con ruido. Muchas gracias."

---

## Preguntas probables

**¿Por qué no hay transición de fase?**
Una escalera de dos piernas es casi unidimensional. Para cualquier *U* > 0 tiene brecha, y el
estado base se conecta suavemente con el de *U* pequeño. Para tener una transición real habría
que cambiar el modelo, por ejemplo con un potencial alternado (Hubbard iónico) o una
interacción entre vecinos *V* (Hubbard extendido).

**¿Para qué sirve el término μ del Hamiltoniano?**
Es el potencial químico. Lo fijamos en μ = *U*/2 para estar a llenado medio (3 electrones ↑
y 3 ↓ en 6 sitios), que es la forma estándar del modelo en ese régimen. Como trabajamos
siempre con el número de partículas fijo, μ·*N* es solo una constante: desplaza las energías,
pero no cambia el estado base, la dinámica ni las brechas. Además simplifica el circuito: con
μ = *U*/2, los términos de un qubit de la interacción se cancelan exactamente y en cada sitio
queda una sola rotación Z↑Z↓.
- *¿Y si lo quitan?* Los resultados serían los mismos (doble ocupación, correlaciones, Δ_c,
  Δ_s). Solo cambia el valor absoluto de la energía, y ED y VQE usan el mismo *H*.
- *¿Por qué no afecta Δ_c si ahí cambia N?* En Δ_c = E(N+1) + E(N−1) − 2E(N), el término μ se
  cancela: μ(N+1) + μ(N−1) − 2μN = 0.
- *¿La cancelación en el circuito es necesaria?* No es esencial. Sin ella, esos términos Z
  suman algo proporcional a *N*, que se conserva, así que serían solo una fase global. Sirve
  para ahorrar compuertas.

**¿Qué significan la brecha de carga y la de espín?**
"La brecha de carga dice cuánto cuesta mover un electrón; la de espín, cuánto cuesta voltear un
espín. En un aislante de Mott la carga queda congelada (Δ_c grande), pero el espín puede seguir
siendo barato. Por eso se miden por separado."

**¿Cómo calcularon las brechas?**
"Las calculamos con diagonalización exacta: buscamos la energía mínima en sectores con un
electrón más o menos para la brecha de carga, y con un espín volteado para la de espín, y
restamos. Cada sector tiene entre 225 y 400 estados, así que es instantáneo. Con VQE se podría
hacer igual, pero su error en la energía sería mayor que las brechas mismas."

**¿Cómo funciona el mapeo de Jordan-Wigner?**
"Jordan-Wigner asigna un qubit a cada modo y reproduce el signo fermiónico con cadenas de Z. En
2D algunos enlaces quedan lejos en el orden y pagan cadenas más largas; en nuestra escalera, las
más largas son de dos qubits, en los peldaños."

**¿Por qué son 9 capas por paso de Trotter?**
"Dentro de una capa todos los saltos deben conmutar, es decir, no compartir sitios. Cada pierna
es un anillo de 3 sitios, un triángulo, donde cualquier par de enlaces comparte un vértice: 3
capas. Los peldaños caben en una. Eso da 4 por espín, 8 con los dos espines, más una de
interacción: 9."

**Si son 9 capas, ¿por qué el circuito tiene 18 cajas?**
"Una capa es un grupo de compuertas que conmutan y se aplican juntas. Las 9 capas contienen 18
saltos, uno por enlace y espín, más 6 interacciones, una por sitio."

**¿Qué son el singlete y el triplete?**
"Dos espines pueden sumar su espín, que es el triplete, o cancelarlo, que es el singlete. Con
interacción débil, nuestra red tiene un nivel a medio llenar y los electrones se alinean, como en
la regla de Hund: el estado base es un triplete. Con interacción fuerte prefieren espines
opuestos con su vecino de peldaño y forman singletes. En *U/t* ≈ 4.3 se cruzan: ese es el salto
que se ve en la gráfica."

**¿Por qué 3×2 y no 4×2?**
Por el VQE: a 16 qubits no convergía ni en *U* = 0 (−5.28 contra −12.00), y un optimizador
con gradiente tardaba 24 minutos por corrida sin cerrar la brecha. Para comparar el mismo
sistema con los tres métodos, cambiamos todo a 3×2.

**Si el esquema es de primer orden, ¿por qué la pendiente es 2?**
"Primer orden" describe el error de cada paso. Para un observable cualquiera, el error total
crece como Δt. Pero el término principal del error es una integral de conmutadores entre el
estado y el observable, y para combinaciones particulares puede anularse. Lo medimos en dos
redes y dio prácticamente igual (1.96 y 1.98). No tenemos una prueba analítica.

**¿Por qué Trotter funciona y VQE no?**
Trotter no optimiza nada: aplica una secuencia fija de compuertas cuyo error controlamos con
Δt. VQE depende de que un optimizador clásico encuentre el mínimo global en un paisaje de 48
dimensiones, y COBYLA es un optimizador local.

**¿Qué es el salto en *U/t* ≈ 4.3? ¿No es un error?**
Es un cruce de niveles. El nivel de un electrón con energía 0 es doble y a llenado medio queda
semi-lleno, así que a *U* pequeño el estado base es un triplete (espín 1). A *U/t* ≈ 4.3 lo
supera un singlete. Lo verificamos con las energías mínimas de los sectores con *S_z* = 0, 1 y 2:
por debajo del cruce, las de 0 y 1 son idénticas.

**¿Correría esto en hardware real?**
Todavía no lo probamos: todo es simulación ideal, sin ruido. Cada paso de Trotter tiene 9 capas
con cadenas de Z, y 50 pasos son un circuito profundo para el hardware actual. Sería el
siguiente paso, empezando por pocos pasos y *U* grande.

**¿Por qué Qiskit y no guppylang/Selene como en el estudio de TFIM?**
Para este modelo queríamos construir a mano la transformación de Jordan-Wigner y las capas de
Trotter, y usar la librería solo para las compuertas y la simulación. Qiskit Aer, además, es
mucho más rápido a este tamaño: el notebook completo corre en menos de 10 minutos.

---

## Datos de respaldo (por si preguntan números)

| Dato | Valor |
|---|---|
| Dimensión del sector a llenado medio | 400 (espacio de Hilbert completo: 2¹² = 4096) |
| Niveles de una partícula (3×2) | {−3, −1, 0, 0, 2, 2} |
| Enlaces | 6 de pierna + 3 de peldaño = 9; 18 términos de salto (dos espines) |
| Capas de Trotter | 9 (3×2) contra 7 (4×2) |
| Parámetros del quench | *T* = 5, 50 pasos (Δt = 0.1), 2000 shots, *U/t* ∈ {1, 4, 8} |
| Costo del circuito | 148 CNOT por paso (profundidad 104, compilado a {CX, RZ, SX, X}) → ~7400 CNOT en 50 pasos |
| Error de Trotter (*U/t* = 4) | 0.026 (5 pasos) → 0.00025 (160 pasos); pendiente 1.96 (4×2: 1.98) |
| VQE | 2 capas, 48 parámetros, COBYLA, 3 reinicios × 600 iteraciones, ~75 s por *U* |
| Energía VQE vs. exacta | *U*=0: −7.03 / −8.00 · *U*=2: −10.21 / −11.59 · *U*=4: −14.33 / −15.79 · *U*=8: −25.32 / −26.31 |
| Tiempo de cómputo total | menos de 10 minutos en una laptop |

**Limitaciones que conviene mencionar si preguntan:**

- Simulador ideal, sin ruido de hardware.
- Solo Trotter de primer orden.
- Los análisis de error de Trotter y de VQE son exactos, sin shots; solo la sección 3 incluye
  ruido de medición.
- Con 6 sitios nada se extrapola al límite termodinámico.
