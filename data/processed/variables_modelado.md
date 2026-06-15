# Variables utilizadas en el modelado

## Relación con `concentradohogar_procesado.csv`

Este documento es un **complemento** de
`diccionario_concentradohogar_procesado.md`: ese archivo describe las 17
columnas del CSV procesado, mientras que aquí se precisa **cuáles de esas
columnas entran realmente a los modelos** (notebook
`03_modelado.ipynb`) y con qué papel (predictora `X` o variable respuesta
`y`), ya que no todas las columnas del CSV se usan como predictores.

De las 17 columnas de `concentradohogar_procesado.csv`:

- **2 son identificadores** (`folioviv`, `foliohog`) y se excluyen siempre
  del modelado — no aportan información predictiva, solo sirven para
  identificar el registro.
- **2 son variables respuesta** (`clase_ingreso_15k`,
  `clase_ingreso_mediana`) — en cada corrida del modelo se usa **una sola**
  como `y`, y la otra se excluye de `X` (función `preparar_X_y` del
  notebook 03).
- **Las 13 restantes son predictoras** y conforman `X` en ambos umbrales —
  el conjunto de predictores es idéntico sin importar cuál variable
  respuesta se esté modelando.

## Conjunto de predictores `X` (13 variables)

| # | Variable | Tipo | Rol en el modelo |
|---|---|---|---|
| 1 | `edad_jefe` | numérica continua | Predictora — se estandariza para Regresión Logística y Red Neuronal |
| 2 | `tasa_dependencia` | numérica continua | Predictora — se estandariza para Regresión Logística y Red Neuronal |
| 3 | `nivel_urbano` | numérica ordinal (1–4) | Predictora |
| 4 | `educa_num` | numérica ordinal (1–11) | Predictora |
| 5 | `es_mujer_jefa` | binaria (0/1) | Predictora |
| 6 | `est_socio_num` | numérica ordinal (1–4) | Predictora — analizada también en el escenario "sin est_socio_num" |
| 7 | `region_Centro_Norte` | binaria (0/1, dummy) | Predictora |
| 8 | `region_Norte` | binaria (0/1, dummy) | Predictora |
| 9 | `region_Sur` | binaria (0/1, dummy) | Predictora |
| 10 | `clase_hog_2` | binaria (0/1, dummy) | Predictora |
| 11 | `clase_hog_3` | binaria (0/1, dummy) | Predictora |
| 12 | `clase_hog_4` | binaria (0/1, dummy) | Predictora |
| 13 | `clase_hog_5` | binaria (0/1, dummy) | Predictora |

La descripción de qué representa cada una de estas variables en el contexto
del problema (ingreso del hogar, estructura familiar, región, etc.) está en
`diccionario_concentradohogar_procesado.md` — aquí no se repite, solo se
indica su rol dentro del pipeline de modelado.

## Variable respuesta `y` (una de las dos, según la corrida)

| Variable | Tipo | Notas |
|---|---|---|
| `clase_ingreso_15k` | binaria (0/1) | Desbalance moderado (~38% / 62%) |
| `clase_ingreso_mediana` | binaria (0/1) | Prácticamente balanceada (~50% / 50%) |

## Categorías base (no representadas explícitamente en `X`)

Por la codificación one-hot con `drop_first=True`, dos categorías quedan
"implícitas" cuando todas las dummies de su grupo valen 0:

- `clase_hog_2`=`clase_hog_3`=`clase_hog_4`=`clase_hog_5`=0 → hogar
  **Unipersonal**.
- `region_Centro_Norte`=`region_Norte`=`region_Sur`=0 → región **Centro**.

Esto es relevante al interpretar los coeficientes de la Regresión Logística:
representan el efecto **respecto a estas categorías base**.

## Escalado de variables

- **Regresión Logística** y **Red Neuronal (MLP)**: las 13 variables de `X`
  se estandarizan (media 0, desviación 1) con `StandardScaler`, ajustado
  únicamente con el conjunto de entrenamiento.
- **Random Forest**: se entrena con las 13 variables en su escala original
  (sin estandarizar), ya que los árboles de decisión no son sensibles a la
  escala de las variables.
