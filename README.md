# Clasificación de hogares mexicanos conforme a su nivel de ingreso mensual

Proyecto de aprendizaje supervisado que evalúa si es posible estimar, a
partir de **variables sociodemográficas básicas** (sin comprobantes
formales de ingreso ni índices socioeconómicos precalculados), si un hogar
mexicano supera un determinado umbral de ingreso mensual. Se utilizan los
microdatos de la **Encuesta Nacional de Ingresos y Gastos de los Hogares
(ENIGH) 2024** publicados por el INEGI.

**Autores**: Diego Morales Ripalda, Juan Carlos Cruz Bravo, Daria Lisset
Landa Apolinar — Facultad de Estadística e Informática, Universidad
Veracruzana.

## Resumen del proyecto

Se trabajó con la tabla `Concentradohogar` (Nueva Serie) de la ENIGH 2024
(91,414 hogares, sin valores nulos ni duplicados). A partir del ingreso
corriente mensual se construyeron dos variables respuesta binarias:

- **Umbral de $15,000 MXN** (≈ 2 salarios mínimos generales 2024, CONASAMI)
- **Mediana muestral** (≈ $18,555 MXN, muy cercana a la Línea de Pobreza por
  Ingresos urbana de CONEVAL)

Se entrenaron y compararon tres algoritmos de clasificación supervisada
(Regresión Logística, Random Forest y una Red Neuronal tipo Perceptrón
Multicapa), validados mediante partición 80/20 estratificada y validación
cruzada (k = 5).

### Resultados principales

| Variable respuesta | Mejor modelo | Accuracy | AUC |
|---|---|---|---|
| $15,000 MXN/mes (≈ 2 salarios mínimos, CONASAMI) | Red Neuronal (MLP) | 0.747 | 0.814 |
| Mediana muestral (≈ LPI urbana, CONEVAL) | Red Neuronal (MLP) | 0.730 | 0.810 |

La Red Neuronal obtuvo el mejor desempeño en ambos umbrales (AUC ≈ 0.81),
seguida de la Regresión Logística (AUC ≈ 0.80); Random Forest mostró el
desempeño más bajo (AUC ≈ 0.76). La estructura del hogar y el nivel
educativo del jefe(a) de hogar resultaron los predictores más relevantes en
los dos modelos con mejor desempeño. Incluir el estrato socioeconómico
construido por el INEGI (`est_socio_num`) aporta una mejora marginal o nula,
lo que respalda la viabilidad de un perfilamiento basado únicamente en
variables demográficas básicas.

El artículo completo, con metodología, resultados y conclusiones, está en
[`reports/Articulo_Capacidad_Pago_ENIGH2024.docx`](reports/Articulo_Capacidad_Pago_ENIGH2024.docx).

## Estructura del repositorio

```
.
├── data/
│   ├── raw/
│   │   └── catalogos/           # Catálogos de códigos del INEGI
│   └── processed/
│       ├── diccionario_concentradohogar_procesado.md
│       └── variables_modelado.md
├── docs/
│   └── diccionario_datos_concentradohogar_enigh2024_ns.csv
├── notebooks/
│   ├── 01_eda_concentradohogar.ipynb
│   ├── 02_preprocesamiento_feature_engineering.ipynb
│   └── 03_modelado.ipynb
├── reports/
│   ├── Articulo_Capacidad_Pago_ENIGH2024.docx   # Artículo final
│   ├── documentacion_metodologia_resultados.docx
│   ├── resumen_modelado.md
│   └── figures/
├── scripts/                       # Scripts de generación de los .docx
├── requirements.txt
└── README.md
```

## Cómo reproducir el análisis

1. **Crear entorno e instalar dependencias**

   ```bash
   python3 -m venv venv
   source venv/bin/activate      # En Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. **Obtener los datos crudos**

   Descargar `conjunto_de_datos_concentradohogar_enigh2024_ns.csv` desde el
   sitio del INEGI:
   <https://www.inegi.org.mx/programas/enigh/nc/2024/#datos_abiertos>

   y colocarlo en `data/raw/conjunto_de_datos_concentradohogar_enigh2024_ns.csv`
   (este archivo no se incluye en el repositorio por su tamaño).

3. **Ejecutar los notebooks en orden**

   - **`01_eda_concentradohogar.ipynb`** — análisis exploratorio: nulos,
     duplicados, distribuciones, valores atípicos, correlaciones y
     definición de los umbrales candidatos para la variable objetivo.

   - **`02_preprocesamiento_feature_engineering.ipynb`** — construye las
     variables respuesta (`clase_ingreso_15k`, `clase_ingreso_mediana`),
     codifica variables categóricas (`region`, `clase_hog`) y crea las
     variables derivadas (`tasa_dependencia`, `nivel_urbano`, `educa_num`,
     `es_mujer_jefa`). Genera el dataset procesado para modelado.

   - **`03_modelado.ipynb`** — entrena y evalúa los tres algoritmos,
     organizados por modelo con hiperparámetros editables, e incluye
     búsqueda de hiperparámetros (`GridSearchCV` / `RandomizedSearchCV`)
     para Random Forest y la Red Neuronal. Genera las tablas comparativas y
     las figuras finales (matrices de confusión, curvas ROC, importancia de
     variables).

## Fuente de datos

INEGI. *Encuesta Nacional de Ingresos y Gastos de los Hogares (ENIGH) 2024,
Nueva Serie.* Conjunto de datos `Concentradohogar` (principales variables
por hogar). Periodo de levantamiento: 21 de agosto al 28 de noviembre de
2024. Tamaño de muestra: 105,718 viviendas particulares a nivel nacional.

## Líneas futuras

- Incorporar índices oficiales de marginación (CONEVAL) a nivel municipal
  mediante `ubica_geo`.
- Explorar algoritmos de boosting (p. ej. Gradient Boosting) con ajuste de
  hiperparámetros.
- Investigar el sesgo de Random Forest hacia `edad_jefe` (variable
  continua de alta cardinalidad con correlación prácticamente nula con el
  ingreso), posiblemente vía `max_features`, `min_samples_leaf` o
  discretización de la variable.
- Replicar el análisis con ediciones posteriores de la ENIGH para evaluar
  la estabilidad temporal de los resultados.
