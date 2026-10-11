# TFM - Imputación de precipitación diaria mediante Machine Learning y componente espacial

## Descripción

Este repositorio contiene el desarrollo del Trabajo Fin de Máster:

**“Evaluación de la influencia del componente espacial en la estimación de datos faltantes de precipitación diaria mediante modelos de Machine Learning aplicados a estaciones meteorológicas”.**

El objetivo del trabajo es evaluar si la incorporación de información espacial mejora la estimación de datos faltantes de precipitación diaria.

Para ello se comparan métodos tradicionales de imputación con modelos de Machine Learning, utilizando registros históricos de estaciones meteorológicas del IDEAM.

---

## Objetivo

Comparar el desempeño de diferentes métodos para la estimación de datos faltantes de precipitación diaria y analizar el efecto de incorporar explícitamente información espacial entre estaciones meteorológicas.

Los métodos evaluados son:

1. Media aritmética vecinal.
2. IDW (Inverse Distance Weighting).
3. Random Forest base.
4. Random Forest con representación espacial mediante IDW.

El desempeño se evalúa utilizando las métricas:

- MAE — Mean Absolute Error.
- RMSE — Root Mean Squared Error.

---

## Fuente de datos

Los registros utilizados corresponden a estaciones meteorológicas del **IDEAM**.

La información del catálogo de estaciones se utiliza para realizar la selección espacial inicial.

Los registros históricos de precipitación diaria fueron descargados manualmente desde la plataforma **DHIME**.

---

## Flujo de trabajo

El proyecto se desarrolla mediante cuatro notebooks consecutivos:

### Notebook 01 — Selección de estaciones

- Carga del catálogo de estaciones.
- Definición de un punto de interés.
- Análisis espacial de proximidad.
- Selección de estaciones candidatas.
- Exportación de la información geográfica para las etapas posteriores.

### Notebook 02 — Preparación y selección de series de precipitación

- Importación de los registros de precipitación descargados desde DHIME.
- Limpieza y estandarización de los datos.
- Construcción de series diarias continuas.
- Identificación de valores faltantes naturales.
- Evaluación de cobertura temporal y solapamiento entre estaciones.
- Análisis de distancia y diferencia de altitud.
- Selección de la estación objetivo, estaciones auxiliares y periodo de estudio.

La configuración finalmente seleccionada fue:

- **Estación objetivo:** 21205710
- **Auxiliar 1:** 21201230
- **Auxiliar 2:** 21201600
- **Periodo de estudio:** 2004–2015

### Notebook 03 — Diseño experimental

- Identificación de valores faltantes naturales.
- Definición de observaciones elegibles.
- Separación temporal en `train`, `valid` y `test`.
- Creación de valores faltantes artificiales.
- Conservación de los valores reales para evaluación.
- Preparación del dataset experimental.

Los valores faltantes artificiales permiten evaluar los métodos de imputación sobre datos cuyo valor real es conocido.

### Notebook 04 — Modelado y evaluación

Se implementan y comparan cuatro métodos:

- Media vecinal.
- IDW.
- Random Forest base.
- Random Forest con componente espacial.

Los modelos se entrenan utilizando información temporal y registros de las estaciones auxiliares.

La evaluación final se realiza exclusivamente sobre el conjunto de prueba (`test`).

---

## Estructura del repositorio

```text
TFM-Precipitacion/
│
├── data/
│   ├── raw/
│   │   └── precipitacion/
│   │
│   ├── intermediate/
│   │
│   └── processed/
│
├── notebooks/
│   ├── 01_Seleccion_Estaciones_DHIME.ipynb
│   ├── 02_Descarga_Precipitaciones_DHIME.ipynb
│   ├── 03_Diseno_Experimental_Imputacion.ipynb
│   └── 04_Imputación_modelado_y_evaluación_de_precipitación_diaria.ipynb
│
├── requirements.txt
├── .gitignore
└── README.md 
