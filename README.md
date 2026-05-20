# Movilidad Social y Educación de Élite: Análisis del Programa "Ser Pilo Paga" en Colombia

**Équipo:** Avril Salazar · Paulina Garza · Othoniel Reyes  
**Curso:** Métodos Analíticos — ITAM, Semestre 2026-I  
**Profesor:** Dr. Jorge Francisco de la Vega Góngora

---

## ¿Qué vamos a hacer?

Reproducimos y extendemos el paper:

> Londoño-Vélez, J., Rodriguez, C., Sanchez, F., & Álvarez-Arango, L. E. (2025). *Financial Aid and Upward Mobility: Evidence from Colombia's Ser Pilo Paga*. NBER Working Paper 31737.

El programa **Ser Pilo Paga (SPP)** otorgó becas completas a estudiantes colombianos de bajos ingresos que superaron un umbral en el examen Saber 11 y se inscribieron en universidades acreditadas de alta calidad. El diseño genera una **discontinuidad precisa/difusa** en el umbral de elegibilidad, lo que permite estimar el efecto causal de acceder a una universidad de élite sobre la movilidad socioeconómica.

**Pregunta central:** ¿Acceder a una universidad de alta calidad a través de una beca aumenta la movilidad socioeconómica de estudiantes de bajos ingresos? ¿Varía este efecto según género, etnia y estrato?

---

## Conocimientos importantes

Esta sección explica toda la teoría necesaria para entender las fases, pruebas y resultados del proyecto. 

---

### 1. Inferencia causal: de asociación a causalidad

**¿Qué es?**  
La mayoría de la estadística clásica mide *asociación* entre variables (correlación). La **inferencia causal** va un paso más allá: busca estimar qué hubiera pasado si las mismas unidades hubieran recibido un tratamiento diferente. A esto se le llama **predicción contrafactual**.

Por ejemplo: ¿cuánto ganarían los becarios de SPP si *no* hubieran recibido la beca? Esa pregunta no se puede responder observando directamente los datos — nunca vemos los dos mundos posibles para la misma persona al mismo tiempo.

**Marco de resultados potenciales (Rubin–Neyman)**  
Para cada individuo $i$ definimos:
- $Y_i(1)$: resultado *si* recibe el tratamiento (inscripción en universidad de élite)
- $Y_i(0)$: resultado *si no* recibe el tratamiento

Solo observamos uno de los dos. El efecto causal individual es $\tau_i = Y_i(1) - Y_i(0)$, que nunca se puede medir directamente.

Los parámetros de interés son promedios sobre la población:

| Parámetro | Definición | Lectura |
|-----------|-----------|---------|
| **ATE** | $E[Y(1) - Y(0)]$ | Efecto promedio para *toda* la población |
| **ATT** | $E[Y(1) - Y(0) \mid D=1]$ | Efecto promedio para quienes *sí* fueron tratados |
| **LATE** | $E[Y(1) - Y(0) \mid \text{Cumplidores}]$ | Efecto promedio para los *cumplidores* (ver §3) |

**¿Por qué importa para SPP?**  
El LATE es el parámetro que estima nuestro diseño Fuzzy RDD. No estimamos el efecto para todos los colombianos ni para todos los becarios — solo para quienes están alrededor del umbral del examen y que se inscribirían en universidad de élite si y solo si obtienen la beca.

**Referencia:** `MA_T1.pdf`

---

### 2. Diagramas causales (DAGs)

**¿Qué es?**  
Un **DAG** (*Directed Acyclic Graph*, gráfica acíclica dirigida) es una herramienta visual para representar relaciones causales entre variables. Los nodos son variables; las flechas indican causalidad ($A \to B$ significa "A causa B").

Los DAGs son útiles para dos cosas:
1. **Identificar confusores**: variables que abren "rutas de puerta trasera" entre el tratamiento y el resultado, sesgando la estimación
2. **Decidir qué controlar**: el **criterio de puerta trasera** dice qué variables deben controlarse para eliminar el sesgo por confusión

**DAG del Fuzzy RDD en SPP:**

```
X (puntaje) → I[X > umbral] → T (elegibilidad)
                                    ↓
                         D (inscripción en élite) → Y (movilidad)
     U (confusores) - - - - - - - - ↑ - - - - - - - - - ↑
```

En el Fuzzy RDD, la indicadora $I[X > x_0]$ actúa como **instrumento** para $D$ (ver §3). Los confusores $U$ no observados afectan $D$ y $Y$, pero *no* $I[X > x_0]$, lo que da validez al instrumento.

**Referencia:** `MA_T2b_DAGs.pdf`

---

### 3. Variables Instrumentales (VI/IV)

**¿Qué es?**  
Las **Variables Instrumentales** son una solución cuando el tratamiento $Z$ no fue asignado al azar y hay confusores no observados $U$ que afectan tanto a $Z$ como a $Y$. En lugar de estimar directamente el efecto de $Z$ en $Y$, se usa una tercera variable $V$ llamada **instrumento** que:

- **(P1) Relevancia:** $V$ tiene un efecto causal en $Z$ (el instrumento mueve el tratamiento)
- **(P2) Restricción de exclusión:** $V$ afecta a $Y$ *únicamente* a través de $Z$ (no hay caminos directos)
- **(P3) No confusión instrumental:** $V$ no está correlacionado con los confusores $U$

Con estos tres supuestos se puede estimar el efecto causal. Un cuarto supuesto es necesario para interpretar el estimador como un promedio con sentido:

- **(P4) Monotonicidad:** Si $V$ aumenta, ningún individuo *reduce* su tratamiento. No hay **desafiadores** (personas que hacen lo contrario de lo que les indica el instrumento).

**Los cuatro estratos principales:**

| Estrato | Símbolo | $V = 0$ | $V = 1$ | Descripción |
|---------|---------|---------|---------|-------------|
| **Cumplidores** | C | $Z = 0$ | $Z = 1$ | Solo se tratan si el instrumento lo indica |
| **Desafiadores** | D | $Z = 1$ | $Z = 0$ | Hacen lo contrario del instrumento |
| **Siempre tratados** | S | $Z = 1$ | $Z = 1$ | Se tratan sin importar el instrumento |
| **Nunca tratados** | N | $Z = 0$ | $Z = 0$ | Nunca se tratan sin importar el instrumento |

Bajo monotonicidad (P4), no hay desafiadores. El estimador IV identifica el **LATE** = efecto causal *para los cumplidores*:

$$\tau_C = E[Y(1) - Y(0) \mid Z(V=1) = 1,\ Z(V=0) = 0]$$

**Estimación práctica — el Ratio de Wald:**

$$\widehat{LATE} = \frac{\text{Forma reducida}}{\text{Primera etapa}} = \frac{E[Y \mid V=1] - E[Y \mid V=0]}{E[Z \mid V=1] - E[Z \mid V=0]}$$

En nuestro proyecto:
- **Instrumento $V$:** $T$ = elegibilidad (estar arriba del umbral en el Saber 11)
- **Tratamiento $Z$:** $D$ = inscripción real en universidad de élite
- **Resultado $Y$:** índice de movilidad socioeconómica

**Referencia:** `MA_T4_1_IV.pdf`

---

### 4. Regresión Discontinua (RDD)

**¿Qué es?**  
El **Diseño de Regresión Discontinua** (RDD) es un método cuasi-experimental que explota el hecho de que el tratamiento cambia *discontinuamente* alrededor de un umbral en una variable continua llamada **variable forzosa** (o *running variable*).

**Idea fundamental:** alrededor del umbral, estudiantes con puntajes ligeramente por encima y por debajo son muy similares en todo (características observables y no observables) — *excepto* en si reciben el tratamiento. Esto hace que la comparación sea *casi tan buena como un experimento aleatorizado*.

**Terminología clave:**

| Término | Definición |
|---------|-----------|
| **Variable forzosa** (*running variable*) $X$ | Variable continua que determina la elegibilidad (puntaje Saber 11 centrado en 0) |
| **Umbral** (*cutoff*) $x_0$ | Valor de $X$ donde el tratamiento cambia (en SPP: $x_0 = 0$) |
| **Ancho de banda** (*bandwidth*) $h$ | Ventana alrededor del umbral que se usa para la estimación |
| **Discontinuidad** | Salto en $P(D=1)$ o en $E(Y)$ exactamente en $x_0$ |

**Dos tipos de RDD:**

**SRD — Regresión Discontinua Precisa (*Sharp*):**  
El tratamiento cambia de 0 a 1 de manera determinista al cruzar el umbral:
$$P(Z = 1 \mid X = x_0^+) = 1, \quad P(Z = 1 \mid X = x_0^-) = 0$$
Todo el que supera el umbral recibe el tratamiento. No hay cumplidores parciales.

**FRD — Regresión Discontinua Difusa (*Fuzzy*):**  
El tratamiento *aumenta* al cruzar el umbral, pero no de manera perfecta:
$$P(Z = 1 \mid X = x_0^+) > P(Z = 1 \mid X = x_0^-)$$
En SPP: ser elegible aumenta la probabilidad de inscribirse en universidad de élite, pero no todos los elegibles se inscriben ni todos los no elegibles dejan de inscribirse.

**El Fuzzy RDD se estima con IV:**  
La indicadora $T = I[X > x_0]$ (elegibilidad) actúa como instrumento para $D$ (inscripción):

$$\widehat{LATE} = \frac{\underbrace{E[Y \mid X = x_0^+] - E[Y \mid X = x_0^-]}_{\text{Forma reducida (ITT en } Y)}}{\underbrace{E[D \mid X = x_0^+] - E[D \mid X = x_0^-]}_{\text{Primera etapa (ITT en } D)}}$$

**Estimación práctica con `rdrobust`:**  
Se ajusta una **regresión local polinomial** a cada lado del umbral, usando solo las observaciones dentro del ancho de banda óptimo $h^*$ (seleccionado por el criterio CCT de Calonico, Cattaneo y Titiunik):

```r
# En R:
rdrobust(y = Y, x = X1, fuzzy = D)
# coef[3] = estimación bias-corrected robusta (la que usamos)
# se[3], pv[3], ci[3,] = SE, p-valor, IC 95%
# bws[1,1] = ancho de banda h* óptimo
```

**Supuestos a verificar en RDD:**

1. **Continuidad de la densidad:** nadie puede manipular su puntaje para quedar justo arriba del umbral. Se verifica con la **prueba de McCrary** (`rddensity`).
2. **Balance de covariables:** características pre-tratamiento no deben saltar en el umbral. Se verifican con `rdrobust` para cada covariable.
3. **Placebos en umbrales falsos:** si el método funciona, no debería haber saltos en el resultado cuando usamos umbrales inventados donde no hay tratamiento.

**Referencia:** `MA_T5_1_RDD.pdf`

---

### 5. Efectos causales condicionales (CATE)

**¿Qué es?**  
El **CATE** (*Conditional Average Treatment Effect*) es el efecto promedio del tratamiento para un subgrupo específico. En lugar de preguntar "¿cuál es el efecto promedio?", pregunta "¿el efecto varía entre mujeres y hombres? ¿entre estudiantes de distinto estrato?"

**Método en Fuzzy RDD:**  
Corremos `rdrobust` por separado dentro de cada subgrupo. El CATE del subgrupo $G$ es el LATE para los *cumplidores de ese subgrupo*.

**Prueba de diferencias:**  
Para saber si la diferencia entre dos CATEs es estadísticamente significativa, usamos el estadístico $z$:
$$z = \frac{\widehat{LATE}_1 - \widehat{LATE}_2}{\sqrt{SE_1^2 + SE_2^2}} \sim N(0,1)$$

**Caracterización de cumplidores:**  
No todos los subgrupos tienen la misma proporción de cumplidores. Si el subgrupo $G$ tiene una primera etapa más alta que el promedio, está sobre-representado entre los cumplidores. Esto se mide con el **índice relativo de cumplidores**:
$$\text{Índice}_G = \frac{\pi_G}{\pi_{\text{global}}} = \frac{E[D \mid T=1,G] - E[D \mid T=0,G]}{E[D \mid T=1] - E[D \mid T=0]}$$

---

### 6. Análisis de sensibilidad

**¿Por qué es necesario?**  
Los métodos cuasi-experimentales como el Fuzzy RDD dependen de supuestos no verificables directamente (exclusión, monotonicidad). El análisis de sensibilidad pregunta: *¿cuánto tendrían que violarse esos supuestos para cambiar las conclusiones?*

**E-valores (Ding & VanderWeele 2016):**  
El **E-valor** mide qué tan fuerte tendría que ser una fuente de sesgo no observada para anular el efecto estimado. Se calcula a partir del riesgo relativo equivalente:
$$E = \widehat{RR} + \sqrt{\widehat{RR}(\widehat{RR} - 1)}$$
Un E-valor alto significa que el resultado es robusto a confusión de magnitud plausible.

**Sensibilidad a la restricción de exclusión:**  
Si la elegibilidad $T$ tiene un efecto directo $\delta$ en $Y$ (más allá de su efecto a través de $D$), el LATE corregido sería:
$$\widehat{LATE}(\delta) = \frac{\text{RF} - \delta}{\text{FS}}$$
Trazamos cómo cambia el LATE bajo distintos valores de $\delta$.

**Cotas de Lee (truncación post-tratamiento):**  
Si el resultado $Y$ solo se observa para quienes *completan* el tratamiento (una selección post-tratamiento), las **cotas de Lee** dan el rango de efectos posibles bajo el peor y mejor escenario de selección.

**Estratificación principal:**  
Análisis formal de los cuatro estratos (cumplidores, siempre tratados, nunca tratados, desafiadores) y sus implicaciones para la interpretación del LATE.

**Referencia:** `MA_T8_Tutifrutti.pdf`; Ding & VanderWeele (2016)

---

## Estructura del documento final

El reporte seguirá las secciones requeridas por el instructor (máximo 20 páginas sin anexos):

1. **Introducción** — problema, supuestos, contexto y alcance
2. **Datos** — descripción del microset (~23,000 obs.), variables clave y contexto institucional
3. **Métodos** — Fuzzy RDD, CATE, análisis de sensibilidad, estratificación principal
4. **Aplicación** — resultados con gráficas y tablas de resumen interpretadas
5. **Conclusiones** — hallazgos, limitaciones y extensiones futuras
6. **Fuentes** — bibliografía
7. **Anexos** — tablas extensas, salidas de código

> Los resultados deben ser **reproducibles**: fijar semillas (`set.seed(999)`) en toda generación de números aleatorios.

---

## Métodos a implementar

### 1. Regresión Discontinua Difusa (Fuzzy RDD) — método principal
El umbral de elegibilidad de SPP genera un salto en la *probabilidad* de tratamiento (inscribirse en universidad de élite), no una asignación perfecta. Esto requiere usar el umbral como **Variable Instrumental** y estimar el **LATE** (Local Average Treatment Effect) para los *cumplidores*.

- **Supuestos a verificar:** continuidad de la densidad (prueba de McCrary), balance de covariables en el umbral, exclusión del instrumento.
- **Estimación:** regresión local polinomial a ambos lados del umbral; selección óptima del ancho de banda (CCT).
- **Referencia de clase:** `MA_T4_1_IV.pdf`, `MA_T5_1_RDD.pdf`

### 2. Efectos Causales Condicionales (CATE)
Extendemos el análisis original para explorar **heterogeneidad del efecto** según:
- Género (`icfes_female`)
- Origen étnico / minoría subrepresentada (`icfes_urm`)
- Estrato socioeconómico (`icfes_stratum`)

El objetivo es determinar si la movilidad ascendente es uniforme o si el programa beneficia desproporcionadamente a ciertos subgrupos.

### 3. Análisis de Sensibilidad (E-valor)
Aplicamos la metodología de **Ding & VanderWeele** para evaluar cuán grande tendría que ser una variable confusora no observada para anular el efecto estimado. También analizamos la robustez a violaciones de la restricción de exclusión y el problema de selección post-tratamiento.

### 4. Estratificación Principal
Discusión formal de los cuatro estratos principales (cumplidores, siempre tratados, nunca tratados, desafiadores) y sus implicaciones para la validez interna y externa del LATE estimado.

---

## Lista de trabajo

### Fase 1 — Preparación y datos

- [x] Leer el paper completo de Londoño-Vélez et al. (2025) y tomar notas sobre diseño, datos y resultados principales
- [x] Obtener y limpiar el microset de datos; documentar variables, valores faltantes y unidades
- [x] Análisis exploratorio: distribución de la variable forzosa (puntaje Saber 11), histogramas, estadísticas descriptivas por grupos (`01_exploratorio.Rmd`)
- [ ] Redactar borrador de secciones **Introducción** y **Datos** del reporte

### Fase 2 — Validación de supuestos

- [x] Prueba de McCrary (densidad de la variable forzosa en el umbral) + gráfica (`02_supuestos.Rmd`)
- [x] Balance de covariables pre-tratamiento en el umbral (tabla de balance y forest plot)
- [x] Placebos en umbrales falsos; sensibilidad al ancho de banda (`02_supuestos.Rmd`)

### Fase 3 — Estimación principal

- [x] Estimación Fuzzy RDD con `rdrobust`: selección de ancho de banda óptimo, estimación del LATE, intervalos de confianza (`03_fuzzy_rdd.Rmd`)
- [x] Análisis de robustez: diferentes polinomios locales, distintos anchos de banda, kernels alternativos
- [x] Análisis CATE por subgrupos (género, etnia, estrato); tabla de heterogeneidad de efectos (`04_cate.Rmd`)

### Fase 4 — Análisis complementarios

- [x] Análisis de sensibilidad con E-valores (`EValue` en R); curva de exclusión; cotas de Lee (`05_sensibilidad.Rmd`)
- [x] Discusión de estratificación principal: tamaño estimado de los estratos, interpretación del LATE
- [ ] Redactar sección **Métodos** del reporte (con referencias formales a los supuestos)

### Fase 5 — Reporte y presentación

- [ ] Integrar y editar el reporte completo (coherencia de estilo, máx. 20 páginas)
- [ ] Redactar secciones **Aplicación** y **Conclusiones**; revisar que todas las gráficas/tablas sean correctas y no redundantes
- [ ] Preparar código reproducible y limpio; mover salidas extensas a **Anexos**
- [ ] Preparar presentación (slides) y repartir secciones para exponer
- [ ] Ensayo de presentación y sesión de preguntas cruzadas entre el equipo

---

## Estructura del repositorio

```
Proyecto Final/
├── README.md               # este archivo
├── data/
│   └── spp_microset.csv    # Fuente: https://github.com/rdpackages-replication/CIT_2024_CUP
│                           # Archivo original: CIT_2024_CUP_fuzzy.csv
│                           # ⚠️  NO subir al repositorio público
├── R/
│   ├── 01_exploratorio.Rmd  # Análisis exploratorio (EDA)
│   ├── 02_supuestos.Rmd     # Validación de supuestos del RDD
│   ├── 03_fuzzy_rdd.Rmd     # Estimación principal y robustez
│   ├── 04_cate.Rmd          # Efectos heterogéneos (CATE)
│   └── 05_sensibilidad.Rmd  # E-valores, exclusión, estratificación principal
├── reporte/
│   └── reporte_final.Rmd   # documento principal
└── presentacion/
    └── slides.Rmd
```

---

## Notas del curso

Todas las notas están en `C:\Users\pauli\Documents\MetodosAnaliticos\Notas\`:

| Archivo | Tema | Relevancia para el proyecto |
|---------|------|----------------------------|
| `MA_T1.pdf` | T1: Introducción — resultados potenciales, ATE, ATT | Fundamento de toda la inferencia causal; define los parámetros que estimamos |
| `MA_T2.pdf` | T2: Inferencia Neymaniana, ajuste por regresión | Marco formal para estimar efectos; justifica el uso de regresión local |
| `MA_T2b_DAGs.pdf` | T2b: Diagramas causales (DAGs), criterio de puerta trasera | Representa el diseño SPP como DAG; justifica por qué T es un instrumento válido |
| `MA_T2_2_Neyman.pdf` | T2: Inferencia Neymaniana (detalle) | Varianza y pruebas de hipótesis para efectos causales |
| `MA_T2_3_Estratificacion.pdf` | T2: Estimación por estratificación | Base conceptual de los análisis por subgrupo (CATE) |
| `MA_T3_1_Observ.pdf` | T3: Estudios observacionales | Contexto: SPP no es un experimento, exige supuestos adicionales |
| `MA_T3_2_Propensity.pdf` | T3: Score de propensidad | Alternativa al RDD; contraste con el diseño que usamos |
| `MA_T3_3_Trt_continuo.pdf` | T3: Tratamiento continuo | Extensión del marco IV/RDD a dosis variables |
| `MA_T3_4_EDR.pdf` | T3: Estimador doblemente robusto | Robustez ante mis-especificación del modelo de resultado |
| `MA_T4_1_IV.pdf` | T4: Variables Instrumentales — **central** | Define los 4 supuestos IV (P1–P4), los estratos (cumplidores, etc.) y el LATE |
| `MA_T5_1_RDD.pdf` | T5: Regresión Discontinua (SRD y FRD) — **central** | Define variable forzosa, umbral, ancho de banda, SRD vs. FRD; base del diseño |
| `MA_T6_1_DiD.pdf` | T6: Diferencias en diferencias | Método alternativo; útil como punto de comparación |
| `MA_T7_SynCon.pdf` | T7: Control sintético | Método para una sola unidad tratada; contraste metodológico |
| `MA_T8_Tutifrutti.pdf` | T8: Métodos adicionales (E-valores, sensibilidad) | Fundamento del análisis de sensibilidad en `05_sensibilidad.Rmd` |
| `MA2026_I_Syllabus.pdf` | Programa del curso | Cronograma y criterios de evaluación |

---

## Variables del conjunto de datos

| Variable | Tipo | Descripción |
|----------|------|-------------|
| `X1` | Continua | Variable forzosa: puntaje Saber 11 **centrado en 0** (positivo = arriba del umbral) |
| `T` | Binaria | Instrumento: `1` si el estudiante es elegible (X1 ≥ 0) |
| `D` | Binaria | Tratamiento: `1` si se inscribió en universidad de élite acreditada |
| `Y` | Binaria | Resultado: indicador de movilidad socioeconómica |
| `icfes_female` | Binaria | `1` = mujer |
| `icfes_age` | Continua | Edad al momento del examen |
| `icfes_urm` | Binaria | `1` = minoría subrepresentada (URM) |
| `icfes_stratum` | Ordinal | Estrato socioeconómico (1 = más bajo) |
| `icfes_famsize` | Continua | Tamaño del hogar familiar |

**N total:** ~23,132 observaciones

---

## Resultados esperados

Al completar el análisis, esperamos encontrar (consistente con Londoño-Vélez et al. 2025):

- **Primera etapa fuerte:** la elegibilidad (T) incrementa la probabilidad de inscripción (D) en ~55 pp cerca del umbral, con estadístico F >> 10
- **LATE positivo y significativo:** recibir la beca y acceder a universidad de élite aumenta la movilidad socioeconómica entre 0.40 y 0.50 puntos (en probabilidad) para los cumplidores locales
- **Prueba de McCrary:** sin evidencia de manipulación del puntaje alrededor del umbral
- **Balance de covariables:** sin saltos significativos en género, edad, URM, estrato, tamaño familiar en el umbral
- **Heterogeneidad:** posibles diferencias de efecto por género y estrato (a determinar empíricamente)
- **Robustez:** E-valor > 4, LATE se mantiene positivo bajo violaciones moderadas de la restricción de exclusión y bajo las cotas de Lee

---

## Fuentes principales

- Londoño-Vélez, J., Rodriguez, C., Sanchez, F., & Álvarez-Arango, L. E. (2025). *Financial Aid and Upward Mobility: Evidence from Colombia's Ser Pilo Paga*. NBER Working Paper 31737.
- Ding, Peng (2024). *A First Course in Causal Inference*. CRC Press. *(texto principal del curso)*
- Huntington-Klein, Nick (2022). *The Effect*. CRC Press.
- Imbens, G. & Lemieux, T. (2008). Regression discontinuity designs: A guide to practice. *Journal of Econometrics*.
- Calonico, S., Cattaneo, M. D., & Titiunik, R. (2014). Robust nonparametric confidence intervals for RDD. *Econometrica*.
- Ding, P. & VanderWeele, T. J. (2016). Sensitivity analysis without assumptions. *Epidemiology*.
- Lee, D. S. (2009). Training, wages, and sample selection: Estimating sharp bounds on treatment effects. *Review of Economic Studies*.
- Material de clase: Temas T1–T8 (notas del Dr. de la Vega Góngora).
