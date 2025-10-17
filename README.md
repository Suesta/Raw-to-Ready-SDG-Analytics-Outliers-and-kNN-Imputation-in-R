# Raw-to-Ready SDG Analytics: Limpieza, Outliers y kNN Imputation en R

Este proyecto muestra un **flujo de preprocesado reproducible de datos** (end-to-end) sobre indicadores alineados con los **ODS (SDG)**. El objetivo es llevar un CSV “crudo” a un **dataset limpio y listo para análisis/modelado**, documentando las decisiones y resultados en un **informe dinámico RMarkdown** (HTML).

Puntos clave para reclutadores:

* **Estandarización de identificadores y tipos**, normalización de etiquetas y control de duplicados.
* **Detección y tratamiento de outliers** (IQR) en **GINI** y análisis de cola derecha en **CO₂**.
* **Correlaciones** focalizadas en 2018 y **búsqueda de variables más relacionadas con Life Expectancy**.
* **Imputación híbrida**: regla específica para 2000←2001 + **kNN (VIM)** con selección de *top features*.
* **Entrega reproducible**: informe HTML + CSV limpio de salida.

---

## Estructura de la carpeta

```
.
├── Data Dictionary.xlsx                 # Diccionario de variables (fuente)
├── Suesta_fichero_clean.csv             # Dataset limpio final (entregable)
├── Suesta_preproceso.html               # Informe RMarkdown renderizado (entregable)
├── Suesta_preproceso.Rmd                # Código fuente del informe (entregable)
├── Enunciado Actividad 1                # Enunciado de la actividad 
└── WorldSustainabilityDataset.csv       # Dataset original crudo (fuente)
```

> La estructura coincide con la captura compartida y con el formato de entrega de la práctica.

---

## Cómo reproducir el proyecto

**Requisitos**

* R (≥ 4.x) y RStudio
* Paquetes: `tidyverse`, `readxl`, `VIM`, `kableExtra`, `ggplot2`, `knitr`

**Pasos**

1. Clona o descarga el repositorio manteniendo estos nombres de archivo.
2. Abre `Suesta_preproceso.Rmd` en RStudio.
3. Haz *Knit to HTML*.

   * El knit genera/actualiza:

     * `Suesta_preproceso.html` (informe)
     * `Suesta_fichero_clean.csv` (salida limpia)

> Nota de compatibilidad: el CSV original usa formato “inglés” (coma como separador y punto decimal). Por eso se lee con `read.csv()` (no `read.csv2()`).

---

## Metodología (resumen ejecutable en el Rmd)

1. **Lectura y verificación**

   * Lectura del CSV crudo con `read.csv()` y del diccionario (`readxl`).
   * Chequeos iniciales de dimensiones, cabeceras y tipos.

2. **Estructura y renombrado**

   * **Estandarización de 5 columnas clave** con mapeo explícito:

     * `Country.Name → Country`
     * `Country.Code → CountryCode`
     * `Regime.Type..RoW.Measure.Definition. → Regime`
     * `Income.Classification..World.Bank.Definition. → Income`
     * `Region` (o `Continent` si faltara `Region`)
   * Verificación de presencia de `Country`, `CountryCode`, `Regime`, `Income`, `Region`.

3. **Tipos e inconsistencias**

   * Conversión segura a numérico de columnas textuales no-ID (limpieza de vacíos, N/A, comas/puntos decimales).
   * Normalización de categóricas (`CountryCode` en mayúsculas; `Regime`, `Income`, `Region` en Title Case).
   * Comprobación de duplicados por `(CountryCode, Year)` y `(Country, Year)` → **no hay duplicados**.

4. **Outliers**

   * **GINI** (`Gini.index..World.Bank.estimate....SI.POV.GINI`): detección de outliers con **IQR** y z-score auxiliar (`GINI_z`).
   * **CO₂** (`Annual.production.based.emissions...GH.EM.IC.LUF`): distribución con **cola derecha** pronunciada; se deja en escala natural para top emisores (z-score auxiliar `CO2_z`).
   * **Top 10 CO₂ (2018)** correctamente listado (China, USA, India, …).

5. **Correlaciones (2018)**

   * Matriz Pearson con ODS seleccionados: ODS1, ODS2, ODS4 (primaria), ODS10 + GDP.
   * **ODS3 (Salud)**: ranking de variables más correlacionadas con **Life Expectancy** (2018), p.ej. Internet usage, enrollment secundaria, acceso a electricidad, pobreza, etc.

6. **Imputación de valores perdidos (ODS3)**

   * Regla específica **2000 ← 2001** cuando 2000 está ausente (por país).

     * **NA antes**: 191 → **NA después**: 20 → **imputados (2000)**: 171.
   * **kNN (VIM, k=5)** sobre `Life.expectancy...SP.DYN.LE00.IN` usando las **5 variables numéricas más correlacionadas** (top-5) + IDs.

     * **NA antes kNN**: 20 → **NA después kNN**: 0.
   * Se añade flag booleano `Life.expectancy..._imp` indicando filas imputadas por kNN.

7. **Tablas resumen (último año disponible)**

   * Por **Región**: tendencia (**media/mediana**) y dispersión (**sd/MAD**) para **Pobreza, GINI, LifeExp** en 2018.

8. **Export**

   * Se crea `Suesta_fichero_clean.csv` sin columnas auxiliares (`GINI_z`, `CO2_z`).

---

## Resultados destacados

* **Estandarización completa** de identificadores y tipado coherente.
* **Control de outliers** en GINI (8 valores extremos identificados) y **asimetría elevada** en CO₂.
* **Correlaciones 2018** coherentes (pobreza/undernourishment negativas con educación y GDP; positivas entre pobreza y undernourishment).
* **Imputación exhaustiva** de Life Expectancy:

  * Regla 2000←2001 + **kNN** → **0 NA restantes**.
  * Columna flag para trazabilidad de imputaciones.

---

## Archivos principales

* **Suesta_preproceso.Rmd** → fuente con:

  * Pregunta por paso, **código**, **salida** y **explicación breve**, siguiendo el enunciado.
  * Salidas voluminosas reemplazadas por **muestras** (`head`, tablas resumidas), evitando listados de >1 página.

* **Suesta_preproceso.html** → informe renderizado para corrección sin necesidad de ejecutar nada.

* **Suesta_fichero_clean.csv** → **dataset limpio** listo para EDA/ML.

* **WorldSustainabilityDataset.csv** y **Data Dictionary.xlsx** → fuentes originales.

---

## Uso del dataset limpio

Ejemplo en R:

```r
library(readr)
df <- read_csv("Suesta_fichero_clean.csv")
str(df)
```

---

## Tecnologías

* **R / RMarkdown**
* **tidyverse, readxl, VIM, kableExtra, ggplot2**

---

## Autor

Víctor Suesta Arribas

> Proyecto académico orientado a buenas prácticas de **Data Cleaning & Imputation** con enfoque reproducible.

¿Quieres que te lo deje también como **README.md** con emojis/badges o más enfocado a portfolio? Te lo preparo en el estilo que prefieras.
