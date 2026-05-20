# Movilidad Social y Educación de Élite: Análisis del Programa "Ser Pilo Paga" en Colombia

**Équipo:** Avril Salazar · Paulina Garza · Othoniel Reyes  
**Curso:** Métodos Analíticos — ITAM, Semestre 2026-I  
**Profesor:** Dr. Jorge Francisco de la Vega Góngora

---

## ¿Qué vamos a hacer?

Reproducimos y extendemos el paper:

> Londoño-Vélez, J., Rodriguez, C., Sanchez, F., & Álvarez-Arango, L. E. (2025). *Financial Aid and Upward Mobility: Evidence from Colombia's Ser Pilo Paga*. NBER Working Paper 31737.

El programa **Ser Pilo Paga (SPP)** otorgó becas completas a estudiantes colombianos de bajos ingresos que superaron un umbral en el examen Saber 11 y se inscribieron en universidades acreditadas de alta calidad. El diseño genera una **discontinuidad sharp/fuzzy** en el umbral de elegibilidad, lo que permite estimar el efecto causal de acceder a una universidad de élite sobre la movilidad socioeconómica.

**Pregunta central:** ¿Acceder a una universidad de alta calidad a través de una beca aumenta la movilidad socioeconómica de estudiantes de bajos ingresos? ¿Varía este efecto según género, etnia y estrato?

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

> Los resultados deben ser **reproducibles**: fijar semillas (`set.seed()`) en toda generación de números aleatorios.

---

## Métodos a implementar

### 1. Regresión Discontinua Difusa (Fuzzy RDD) — método principal
El umbral de elegibilidad de SPP genera un salto en la *probabilidad* de tratamiento (inscribirse en universidad de élite), no una asignación perfecta. Esto requiere usar el umbral como **Variable Instrumental** y estimar el **LATE** (Local Average Treatment Effect) para los *compliers*.

- **Supuestos a verificar:** continuidad de la densidad (prueba de McCrary), balance de covariables en el umbral, exclusión del instrumento.
- **Estimación:** regresión local polinomial a ambos lados del umbral; selección óptima del ancho de banda (Imbens-Kalyanaraman o CCT).
- **Referencia de clase:** Tema 4 — Variables instrumentales y Tema 5 — RDD (notas `MA_T4_1_IV.pdf`, `MA_T5_1_RDD.pdf`).

### 2. Efectos Causales Condicionales (CATE)
Extendemos el análisis original para explorar **heterogeneidad del efecto** según:
- Género (`icfes_female`)
- Origen étnico / minoría subrepresentada (`icfes_urm`)
- Estrato socioeconómico

El objetivo es determinar si la movilidad ascendente es uniforme o si el programa beneficia desproporcionadamente a ciertos subgrupos.

### 3. Análisis de Sensibilidad (E-value)
Aplicamos la metodología de **Ding & VanderWeele** para evaluar cuán grande tendría que ser una variable confusora no observada para anular el efecto estimado. También verificamos posible **manipulación de la variable forzosa** (sorting alrededor del umbral).

### 4. Estratificación Principal
Discusión formal del **sesgo de selección post-tratamiento**: las variables de resultado (graduación, empleo, salario) solo se observan para quienes completan la carrera, lo cual no es aleatorio. Se analizará el alcance y las limitaciones de este problema en el contexto de SPP.

---

## Conocimiento previo necesario

Antes de comenzar, el equipo debe tener dominio de los siguientes conceptos del curso:

| Tema | Concepto clave | Referencia |
|------|---------------|------------|
| T1 | Marco de resultados potenciales, ATE, ATT | `MA_T1.pdf` |
| T2 | Inferencia Neymaniana, ajuste por regresión | `MA_T2_2_Neyman.pdf` |
| T3 | Score de propensidad, estimador doblemente robusto | `MA_T3_2_Propensity.pdf`, `MA_T3_4_EDR.pdf` |
| T4 | Variables instrumentales, supuestos (relevancia, exclusión, monotonía), LATE | `MA_T4_1_IV.pdf` |
| T5 | Regresión discontinua: sharp vs. fuzzy, ancho de banda, regresión local | `MA_T5_1_RDD.pdf` |

**Herramientas de R requeridas:**
- `rdrobust` — estimación de RDD con ancho de banda óptimo
- `rddensity` / `DCdensity` — prueba de McCrary para manipulación
- `AER` o `ivreg` — estimación IV/2SLS para la versión fuzzy
- `ggplot2` — visualizaciones de la discontinuidad
- `cobalt` o `MatchIt` — balance de covariables
- `EValue` — análisis de sensibilidad

---

## Lista de trabajo

### Fase 1 — Preparación y datos (semana 1)

- [ ] Leer el paper completo de Londoño-Vélez et al. (2025) y tomar notas sobre diseño, datos y resultados principales
- [ ] Obtener y limpiar el microset de datos; documentar variables, valores faltantes y unidades
- [ ] Análisis exploratorio: distribución de la variable forzosa (puntaje Saber 11), histogramas, estadísticas descriptivas por grupos
- [ ] Redactar borrador de secciones **Introducción** y **Datos** del reporte

### Fase 2 — Validación de supuestos (semana 2)

- [ ] Prueba de McCrary (densidad de la variable forzosa en el umbral) + gráfica
- [ ] Balance de covariables pre-tratamiento en el umbral (tabla de balance)
- [ ] Gráfica de discontinuidad: variable de resultado vs. variable forzosa, con ajuste polinomial a cada lado

### Fase 3 — Estimación principal (semana 2–3)

- [ ] Estimación Fuzzy RDD con `rdrobust`: selección de ancho de banda óptimo, estimación del LATE, intervalos de confianza
- [ ] Análisis de robustez: diferentes polinomios locales, distintos anchos de banda, placebo en falsos umbrales
- [ ] Análisis CATE por subgrupos (género, etnia, estrato); tabla de heterogeneidad de efectos

### Fase 4 — Análisis complementarios (semana 3)

- [ ] Análisis de sensibilidad con E-values (`EValue` en R); interpretación en términos del dominio
- [ ] Discusión de estratificación principal: cuantificar el problema de selección post-tratamiento
- [ ] Redactar sección **Métodos** del reporte (con referencias formales a los supuestos)

### Fase 5 — Reporte y presentación (semana 4)

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
├── R/
│   ├── 01_exploratorio.R
│   ├── 02_supuestos.R
│   ├── 03_fuzzy_rdd.R
│   ├── 04_cate.R
│   └── 05_sensibilidad.R
├── reporte/
│   └── reporte_final.Rmd   # documento principal
└── presentacion/
    └── slides.Rmd
```

---

## Fuentes principales

- Londoño-Vélez, J., Rodriguez, C., Sanchez, F., & Álvarez-Arango, L. E. (2025). *Financial Aid and Upward Mobility: Evidence from Colombia's Ser Pilo Paga*. NBER Working Paper 31737.
- Ding, Peng (2024). *A First Course in Causal Inference*. CRC Press. *(texto principal del curso)*
- Huntington-Klein, Nick (2022). *The Effect*. CRC Press.
- Imbens, G. & Lemieux, T. (2008). Regression discontinuity designs: A guide to practice. *Journal of Econometrics*.
- Calonico, S., Cattaneo, M. D., & Titiunik, R. (2014). Robust nonparametric confidence intervals for RDD. *Econometrica*.
- Ding, P. & VanderWeele, T. J. (2016). Sensitivity analysis without assumptions. *Epidemiology*.
- Material de clase: Temas T1–T5 (notas del Dr. de la Vega Góngora).
