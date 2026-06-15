# Estimador de Capacidad de Pago en el Sector Informal (ENIGH 2024)

Proyecto de aprendizaje supervisado para clasificar hogares mexicanos según
su nivel de ingreso mensual, utilizando exclusivamente variables
sociodemográficas y de estructura del hogar (sin comprobantes formales de
ingreso), a partir de los microdatos de la **Encuesta Nacional de Ingresos y
Gastos de los Hogares (ENIGH) 2024** del INEGI.

## Estructura del proyecto

```
.
├── data/
│   ├── raw/                     # Datos originales (NO se versionan, ver .gitignore)
│   │   ├── conjunto_de_datos_concentradohogar_enigh2024_ns.csv
│   │   └── catalogos/           # Catálogos de códigos (sí se versionan)
│   │       ├── clase_hog.csv
│   │       ├── educa_jefe.csv
│   │       ├── est_socio.csv
│   │       ├── sexo.csv
│   │       ├── tam_loc.csv
│   │       └── ubica_geo.csv
│   └── processed/               # Datos limpios / listos para modelar (no versionados)
│       ├── concentradohogar_procesado.csv
│       ├── diccionario_df_ml.csv
│       └── diccionario_concentradohogar_procesado.md
├── docs/
│   └── diccionario_datos_concentradohogar_enigh2024_ns.csv
├── notebooks/
│   ├── 01_eda_concentradohogar.ipynb              # Diagnóstico exploratorio de datos
│   ├── 02_preprocesamiento_feature_engineering.ipynb  # Limpieza y construcción de variables
│   └── 03_modelado.ipynb                          # Entrenamiento y evaluación de modelos
├── reports/
│   ├── resumen_eda.md
│   ├── resumen_modelado.md
│   ├── resultados_modelado_principal.csv
│   ├── resultados_sensibilidad_est_socio.csv
│   ├── resultados_sensibilidad_balanceo.csv
│   ├── documentacion_metodologia_resultados.docx
│   └── figures/                 # Figuras generadas por los notebooks
├── src/                          # Funciones reutilizables (futuro)
├── requirements.txt
└── README.md
```

## Cómo empezar

1. **Crear entorno e instalar dependencias**

   ```bash
   python3 -m venv venv
   source venv/bin/activate      # En Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. **Colocar los datos crudos**

   Descargar `conjunto_de_datos_concentradohogar_enigh2024_ns.csv` desde el
   sitio del INEGI:
   <https://www.inegi.org.mx/programas/enigh/nc/2024/#datos_abiertos>

   y colocarlo en:

   ```
   data/raw/conjunto_de_datos_concentradohogar_enigh2024_ns.csv
   ```

3. **Ejecutar los notebooks en orden**

   ```bash
   jupyter notebook notebooks/
   ```

   - **`01_eda_concentradohogar.ipynb`** — diagnóstico exploratorio: nulos,
     duplicados, distribuciones, correlaciones, definición de los umbrales
     candidatos para la variable objetivo. Genera `reports/resumen_eda.md`
     y figuras en `reports/figures/`.

   - **`02_preprocesamiento_feature_engineering.ipynb`** — construye la
     variable objetivo (`clase_ingreso_15k` y `clase_ingreso_mediana`),
     codifica variables categóricas (`region`, `clase_hog`) y crea variables
     derivadas (`tasa_dependencia`, `nivel_urbano`, `educa_num`,
     `es_mujer_jefa`). Genera `data/processed/concentradohogar_procesado.csv`
     y `data/processed/diccionario_df_ml.csv`. La descripción narrativa de
     cada variable resultante está en
     `data/processed/diccionario_concentradohogar_procesado.md`.

   - **`03_modelado.ipynb`** — entrena y evalúa tres algoritmos (Regresión
     Logística, Random Forest y Red Neuronal/MLP), organizado **por
     modelo** con un bloque de hiperparámetros editable en cada sección,
     más búsqueda de hiperparámetros (`GridSearchCV` /
     `RandomizedSearchCV`) para Random Forest y la Red Neuronal. Al final
     incluye la comparación de los 3 modelos (matrices de confusión, curvas
     ROC, importancia de variables) y los análisis de sensibilidad
     (`est_socio_num` y balanceo de clases). Genera las tablas de
     `reports/resultados_*.csv` y las figuras de `reports/figures/`.

## Resultados principales

| Umbral | Mejor modelo | Accuracy | AUC |
|---|---|---|---|
| $15,000 MXN/mes (≈2 salarios mínimos, CONASAMI) | Red Neuronal (MLP) | 0.747 | 0.814 |
| Mediana muestral (≈LPI urbana, CONEVAL) | Red Neuronal (MLP) | 0.730 | 0.810 |

Un resumen completo de resultados, hallazgos e implicaciones está disponible
en `reports/resumen_modelado.md` y en
`reports/documentacion_metodologia_resultados.docx` (metodología, resultados
y conclusiones para el artículo académico).

## Fuente de datos

INEGI. *Encuesta Nacional de Ingresos y Gastos de los Hogares (ENIGH) 2024,
Nueva Serie.* Conjunto de datos `concentradohogar` (principales variables
por hogar). Periodo de levantamiento: 21 de agosto al 28 de noviembre de
2024. Tamaño de muestra: 105,718 viviendas particulares a nivel nacional.

## Líneas futuras

- Incorporar índices oficiales de marginación (CONEVAL) a nivel municipal
  mediante `ubica_geo`.
- Explorar algoritmos de boosting (p. ej. Gradient Boosting) con ajuste de
  hiperparámetros.
- Investigar por qué Random Forest asigna una importancia
  desproporcionada a `edad_jefe` (variable con correlación prácticamente
  nula con el ingreso): posible sesgo hacia variables continuas de alta
  cardinalidad, a corregir vía `max_features`, `min_samples_leaf` o
  discretización de la variable.
