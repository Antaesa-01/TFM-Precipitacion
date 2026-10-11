# TFM - Imputación de precipitación diaria mediante Machine Learning y componente espacial

## Descripción

Este repositorio contiene el desarrollo del Trabajo Fin de Máster:

**“Evaluación de la influencia del componente espacial en la estimación de datos faltantes de precipitación diaria mediante modelos de Machine Learning aplicados a estaciones meteorológicas”.**

El trabajo busca evaluar si la incorporación de información espacial mejora la estimación de datos faltantes de precipitación diaria en estaciones meteorológicas.

Para ello se comparan métodos tradicionales de imputación con modelos de Machine Learning utilizando registros históricos de precipitación y características espaciales de estaciones meteorológicas del IDEAM.

---

## Objetivo

Comparar el desempeño de diferentes métodos para la estimación de datos faltantes de precipitación diaria y analizar el efecto de incorporar explícitamente información espacial entre estaciones meteorológicas.

Los métodos evaluados son:

1. Media aritmética vecinal.
2. IDW (*Inverse Distance Weighting*).
3. Random Forest base.
4. Random Forest con representación espacial mediante IDW.

El desempeño de los métodos se evalúa mediante:

- **MAE** — Mean Absolute Error.
- **RMSE** — Root Mean Squared Error.

---

## Fuente de datos

Los registros utilizados corresponden a estaciones meteorológicas del **Instituto de Hidrología, Meteorología y Estudios Ambientales de Colombia — IDEAM**.

La información del catálogo de estaciones se utiliza para realizar la selección espacial inicial.

Los registros históricos de precipitación diaria fueron descargados manualmente desde la plataforma **DHIME**.

---

## Flujo de trabajo

El proyecto se desarrolla mediante cuatro notebooks consecutivos.

### Notebook 01 — Selección de estaciones

Se realiza la selección espacial inicial de estaciones meteorológicas a partir de un punto de interés.

Principales actividades:

- Carga del catálogo de estaciones.
- Definición del punto de interés.
- Procesamiento de información espacial.
- Cálculo de distancias.
- Selección de estaciones cercanas.
- Exportación de las estaciones candidatas.

Salida principal:

```text
data/raw/estaciones_candidatas.geojson
```

---

### Notebook 02 — Preparación y selección de series de precipitación

Se integran los registros diarios descargados desde DHIME y se analiza su disponibilidad temporal y espacial.

Principales actividades:

- Importación y consolidación de registros de precipitación.
- Limpieza y estandarización de datos.
- Revisión de registros duplicados.
- Construcción de series diarias continuas.
- Conservación de valores faltantes naturales como `NaN`.
- Evaluación de cobertura temporal.
- Análisis de solapamiento entre estaciones.
- Cálculo de distancias y diferencias de altitud.
- Comparación de configuraciones de estaciones.
- Selección de la estación objetivo, estaciones auxiliares y periodo de estudio.

La configuración seleccionada fue:

- **Estación objetivo:** `21205710`
- **Estación auxiliar 1:** `21201230`
- **Estación auxiliar 2:** `21201600`
- **Periodo de estudio:** 2004–2015

Las principales salidas del Notebook 02 son:

```text
data/intermediate/disponibilidad_periodo_seleccionado.parquet
data/intermediate/precipitacion_periodo_seleccionado.parquet
data/intermediate/estaciones_seleccionadas.geojson
```

---

### Notebook 03 — Diseño experimental

Se construye el experimento que permitirá evaluar objetivamente los métodos de imputación.

Principales actividades:

- Construcción del calendario diario del periodo de estudio.
- Identificación de valores faltantes naturales.
- Definición de observaciones elegibles.
- Separación temporal de los datos.
- Generación de valores faltantes artificiales.
- Conservación de los valores originales como referencia.
- Construcción del conjunto experimental.

La separación temporal utilizada es:

- **Train:** 2004–2011
- **Validation:** 2012–2013
- **Test:** 2014–2015

Los valores faltantes artificiales se generan únicamente sobre observaciones cuyo valor real es conocido. De esta manera, las estimaciones realizadas posteriormente pueden compararse directamente con la precipitación realmente observada.

Las principales salidas son:

```text
data/processed/03_dataset_experimental.parquet
data/processed/03_info_y_verdadera.parquet
data/processed/03_config_experimento.json
```

---

### Notebook 04 — Imputación, modelado y evaluación

Se implementan y comparan los cuatro métodos de imputación considerados en el estudio.

#### Métodos tradicionales

1. Media aritmética vecinal.
2. IDW con exponente `p = 2`.

#### Métodos de Machine Learning

3. Random Forest base.
4. Random Forest con representación espacial mediante `P_IDW`.

El Random Forest base utiliza información de las estaciones auxiliares, indicadores de disponibilidad y variables temporales cíclicas.

El modelo espacial incorpora adicionalmente el valor estimado mediante IDW como representación explícita del componente espacial.

Los hiperparámetros se seleccionan utilizando el conjunto de validación y la evaluación final se realiza sobre el conjunto de prueba.

---

## Diseño experimental

Una parte fundamental del experimento consiste en diferenciar entre dos tipos de valores faltantes:

### Valores faltantes naturales

Son registros que originalmente no se encuentran disponibles en las series meteorológicas.

Estos valores pueden ser imputados, pero no pueden utilizarse para calcular el error del modelo debido a que su valor real es desconocido.

### Valores faltantes artificiales

Son valores reales conocidos que se ocultan intencionalmente durante el experimento.

Posteriormente, cada método intenta estimarlos y la predicción puede compararse con el valor verdadero.

Este procedimiento permite evaluar los modelos mediante MAE y RMSE.

---

## Resultados principales

La evaluación final se realizó sobre **126 observaciones artificialmente ocultadas del conjunto de prueba**.

| Modelo | MAE | RMSE |
|---|---:|---:|
| Media vecinal | 2.179 | 4.312 |
| IDW | 2.217 | 4.340 |
| Random Forest base | 2.636 | 4.039 |
| Random Forest espacial | 2.762 | 4.140 |

Los resultados muestran comportamientos diferentes dependiendo de la métrica utilizada.

- La **media vecinal** presentó el menor MAE.
- El **Random Forest base** presentó el menor RMSE.

Al comparar los métodos equivalentes:

- Media vecinal → IDW:
  - MAE: **+1.77 %**
  - RMSE: **+0.65 %**

- Random Forest base → Random Forest espacial:
  - MAE: **+4.76 %**
  - RMSE: **+2.49 %**

Los valores positivos indican un aumento del error.

Por lo tanto, para la configuración de estaciones, periodo de estudio y representación espacial evaluados, la incorporación explícita del componente espacial mediante IDW no produjo una mejora en el conjunto final de prueba.

Este resultado no implica que la información espacial no sea relevante de manera general, sino que bajo las condiciones específicas del experimento desarrollado no se observó una mejora en la precisión de las estimaciones.

---

## Estructura del repositorio

```text
TFM-Precipitacion/
│
├── data/
│   │
│   ├── raw/
│   │   ├── precipitacion/
│   │   └── estaciones_candidatas.geojson
│   │
│   ├── intermediate/
│   │   ├── disponibilidad_periodo_seleccionado.parquet
│   │   ├── precipitacion_periodo_seleccionado.parquet
│   │   └── estaciones_seleccionadas.geojson
│   │
│   └── processed/
│       ├── 03_dataset_experimental.parquet
│       ├── 03_info_y_verdadera.parquet
│       └── 03_config_experimento.json
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
```

---

## Organización de los datos

### `data/raw`

Contiene los datos originales utilizados en el proyecto.

Incluye:

- Catálogo e información espacial inicial.
- Estaciones candidatas.
- Archivos de precipitación descargados desde DHIME.

Los archivos de esta carpeta representan la información de entrada al proceso de análisis.

### `data/intermediate`

Contiene información generada durante las etapas de selección y preparación de las series.

Estos archivos constituyen principalmente las salidas del Notebook 02 y las entradas del Notebook 03.

### `data/processed`

Contiene los datos preparados específicamente para el experimento de imputación y Machine Learning.

Estos archivos son generados en el Notebook 03 y utilizados posteriormente en el Notebook 04.

---

## Orden de ejecución

Para reproducir el flujo completo del proyecto se recomienda ejecutar los notebooks en el siguiente orden:

```text
Notebook 01
Selección espacial de estaciones
        ↓
Notebook 02
Preparación de series y selección
de estaciones y periodo
        ↓
Notebook 03
Diseño experimental y generación
de valores faltantes artificiales
        ↓
Notebook 04
Imputación, Machine Learning
y evaluación final
```

Cada notebook genera los archivos necesarios para las etapas posteriores.

---

## Instalación

Las principales dependencias se encuentran especificadas en:

```text
requirements.txt
```

Para instalarlas:

```bash
pip install -r requirements.txt
```

Las principales librerías utilizadas son:

- NumPy
- Pandas
- Matplotlib
- GeoPandas
- Shapely
- PyProj
- Contextily
- Scikit-learn
- PyArrow
- Jupyter

---

## Tecnologías utilizadas

El proyecto fue desarrollado principalmente en **Python** mediante notebooks Jupyter/Google Colab.

Se utilizan herramientas de:

- Ciencia de datos.
- Procesamiento de series temporales.
- Sistemas de información geográfica.
- Análisis espacial.
- Machine Learning.
- Visualización de datos.

---

## Consideraciones metodológicas

La descarga de los registros históricos de precipitación desde DHIME se realizó manualmente.

Los valores faltantes naturales se conservaron durante la preparación de las series y no se utilizaron para evaluar directamente los modelos debido a la ausencia de un valor verdadero conocido.

La evaluación se realizó mediante valores artificialmente ocultados para garantizar que las predicciones pudieran compararse con observaciones reales.

La separación de los datos se realizó temporalmente para evitar utilizar información futura durante el entrenamiento de los modelos.

Los resultados obtenidos deben interpretarse dentro de las condiciones específicas del experimento desarrollado, incluyendo:

- Estaciones seleccionadas.
- Disponibilidad de información auxiliar.
- Periodo 2004–2015.
- Número de estaciones auxiliares.
- Representación espacial utilizada.
- Método de imputación evaluado.

---

## Autor

Trabajo Fin de Máster desarrollado como parte del programa de **Máster en Big Data y Ciencia de Datos**.

Universidad Internacional de Valencia — VIU.
