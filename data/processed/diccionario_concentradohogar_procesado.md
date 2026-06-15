# Diccionario de datos — `concentradohogar_procesado.csv`

## Descripción general

Este archivo es el resultado del proceso de limpieza e ingeniería de
características aplicado a la tabla `concentradohogar_enigh2024_ns` del
INEGI (notebook `02_preprocesamiento_feature_engineering.ipynb`). Contiene
**91,414 hogares** (filas) y **17 variables** (columnas), listas para ser
usadas como entrada de los modelos de clasificación supervisada.

Cada fila representa un hogar mexicano encuestado en la ENIGH 2024. El
objetivo del proyecto es estimar, a partir de características puramente
sociodemográficas, si el ingreso mensual de un hogar supera cierto umbral —
es decir, aproximar su **capacidad de pago** sin recurrir a comprobantes
formales de ingreso.

## Variables

### Identificadores

| Variable | Tipo | Descripción |
|---|---|---|
| `folioviv` | texto/numérico | Identificador único de la vivienda dentro de la ENIGH. No se usa como predictor; sirve para dar seguimiento o cruzar con otras tablas del INEGI. |
| `foliohog` | texto/numérico | Identificador del hogar dentro de la vivienda (una vivienda puede tener más de un hogar). Junto con `folioviv` forma la llave única de cada registro. |

### Variables respuesta (target)

| Variable | Tipo | Descripción |
|---|---|---|
| `clase_ingreso_15k` | binaria (0/1) | Indica si el ingreso mensual del hogar (`ing_cor / 3`) es mayor o igual a $15,000 MXN, umbral equivalente a ≈2 salarios mínimos generales 2024 (CONASAMI). 1 = el hogar alcanza o supera ese ingreso; 0 = no lo alcanza. Distribución: ≈38% clase 0 / ≈62% clase 1 (desbalance moderado). |
| `clase_ingreso_mediana` | binaria (0/1) | Igual que la anterior, pero usando como umbral la mediana muestral del ingreso mensual (≈$18,555 MXN), que coincide casi exactamente con la Línea de Pobreza por Ingresos urbana de CONEVAL para un hogar de 4 integrantes. Distribución prácticamente balanceada (≈50% / 50%). |

Ambas variables se construyen a partir de la misma cantidad subyacente
(ingreso mensual), pero representan dos interpretaciones distintas: la
primera como un criterio de "capacidad de pago" tipo crédito (múltiplo del
salario mínimo), y la segunda como un criterio de "vulnerabilidad por
ingresos" tipo política social (línea de pobreza oficial). En el modelado se
entrenan y evalúan modelos independientes para cada una.

### Variables predictoras — características del jefe(a) de hogar

| Variable | Tipo | Descripción |
|---|---|---|
| `edad_jefe` | numérica (entero) | Edad en años cumplidos del jefe o jefa del hogar. Captura efectos de ciclo de vida (ingresos suelen subir y luego bajar con la edad). |
| `educa_num` | numérica ordinal (1–11) | Nivel máximo de estudios del jefe(a) de hogar, recodificado como escala numérica de 1 (sin instrucción) a 11 (posgrado). Es uno de los predictores más fuertes del ingreso del hogar. |
| `es_mujer_jefa` | binaria (0/1) | 1 si el jefe(a) de hogar es mujer, 0 si es hombre. Permite analizar diferencias asociadas al sexo de quien encabeza el hogar (≈32% de los hogares tienen jefatura femenina en la muestra). |

### Variables predictoras — composición y estructura del hogar

| Variable | Tipo | Descripción |
|---|---|---|
| `tasa_dependencia` | numérica (razón, ≥1) | Número de integrantes del hogar dividido entre el número de ocupados (si no hay ocupados, se usa el total de integrantes). Representa la "carga económica" por persona que trabaja: valores altos indican que pocas personas sostienen económicamente a muchas. |
| `clase_hog_2` | binaria (0/1) | 1 si el hogar es de tipo "Nuclear" (jefe(a), cónyuge e hijos, sin otros parientes). Categoría más común (~61% de los hogares). |
| `clase_hog_3` | binaria (0/1) | 1 si el hogar es de tipo "Ampliado" (hogar nuclear más otros parientes, p. ej. abuelos, tíos). ~23% de los hogares. |
| `clase_hog_4` | binaria (0/1) | 1 si el hogar es de tipo "Compuesto" (hogar nuclear o ampliado más personas sin parentesco con el jefe). Categoría poco frecuente (~minoritaria). |
| `clase_hog_5` | binaria (0/1) | 1 si el hogar es de tipo "Corresidente" (dos o más personas sin relación de parentesco que comparten la vivienda y el gasto). La menos frecuente. |

> La categoría base (no representada por ninguna de las 4 columnas
> anteriores) es **"Unipersonal"** (hogar de una sola persona). Esto es el
> resultado de aplicar codificación one-hot con `drop_first=True` sobre la
> variable original `clase_hog` (5 categorías).

### Variables predictoras — entorno geográfico

| Variable | Tipo | Descripción |
|---|---|---|
| `nivel_urbano` | numérica ordinal (1–4) | Grado de urbanización de la localidad donde vive el hogar, construida a partir de `tam_loc` (1 = localidad rural, &lt;2,500 habitantes; 4 = metrópoli, ≥100,000 habitantes). A mayor valor, mayor urbanización. |
| `region_Norte` | binaria (0/1) | 1 si el hogar reside en la región Norte (Baja California, Coahuila, Chihuahua, Nuevo León, Sonora, Tamaulipas), según la regionalización de Banco de México (2024). |
| `region_Centro_Norte` | binaria (0/1) | 1 si el hogar reside en la región Centro-Norte (Aguascalientes, Baja California Sur, Colima, Durango, Jalisco, Michoacán, Nayarit, San Luis Potosí, Sinaloa, Zacatecas). |
| `region_Sur` | binaria (0/1) | 1 si el hogar reside en la región Sur (Campeche, Chiapas, Guerrero, Oaxaca, Quintana Roo, Tabasco, Veracruz, Yucatán). |

> La categoría base es **"Centro"** (CDMX, Estado de México, Guanajuato,
> Hidalgo, Morelos, Puebla, Querétaro, Tlaxcala): un hogar con las tres
> columnas de región en 0 corresponde a esta región. Igual que con
> `clase_hog`, se aplicó codificación one-hot con `drop_first=True` sobre 4
> macrorregiones que cubren las 32 entidades federativas.

### Variable predictora — estrato socioeconómico (uso opcional)

| Variable | Tipo | Descripción |
|---|---|---|
| `est_socio_num` | numérica ordinal (1–4) | Estrato socioeconómico del hogar, construido por el INEGI a partir de características de la vivienda (1 = bajo, 4 = alto). Se incluye como predictor "de referencia", pero los análisis de sensibilidad muestran que el modelo funciona casi igual sin esta variable (diferencia de AUC ≤0.005), lo que respalda el uso de variables puramente demográficas para estimar capacidad de pago. |

## Notas adicionales

- El dataset **no contiene valores nulos ni filas duplicadas** (verificado
  tanto a nivel de fila completa como por la combinación `folioviv` +
  `foliohog`).
- Para Regresión Logística y Red Neuronal, todas las variables numéricas se
  **estandarizan** (media 0, desviación 1) antes de entrenar; Random Forest
  se entrena con las variables en su escala original.
- El diccionario completo de las variables *originales* de la tabla
  `concentradohogar_enigh2024_ns` (antes de la ingeniería de
  características) está disponible en
  `docs/diccionario_datos_concentradohogar_enigh2024_ns.csv`.
