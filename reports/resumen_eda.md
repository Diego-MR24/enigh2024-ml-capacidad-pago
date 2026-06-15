# Resumen Ejecutivo — EDA `concentradohogar` ENIGH 2024

## Panorama general

- Filas: 91,414
- Columnas totales: 126
- Memoria total: 126.41 MB
- Filas completamente duplicadas: 0
- Duplicados por (folioviv, foliohog): 0

## Valores nulos (variables de interés)

- Sin valores nulos en las variables de interés.

## Variable objetivo (ing_mensual)

- Media: 24,091.20 MXN
- Mediana: 18,555.39 MXN
- Desviación estándar: 31,292.20 MXN
- Mínimo: 0.00 MXN
- Máximo: 5,810,659.18 MXN

## Balance de clases por umbral candidato

| umbral                                  |   valor_umbral |   pct_clase_0 (< umbral) |   pct_clase_1 (>= umbral) |
|:----------------------------------------|---------------:|-------------------------:|--------------------------:|
| Mediana                                 |        18555.4 |                    49.99 |                     50.01 |
| 15,000 MXN (umbral propuesto en README) |        15000   |                    38.14 |                     61.86 |
| Percentil 60                            |        22134.4 |                    60    |                     40    |
| Percentil 75                            |        29613.6 |                    75    |                     25    |

## Outliers (criterio IQR)

|             |    Q1 |      Q3 |     IQR |   limite_inferior |   limite_superior |   n_outliers |   pct_outliers |
|:------------|------:|--------:|--------:|------------------:|------------------:|-------------:|---------------:|
| ing_mensual | 11507 | 29613.6 | 18106.6 |          -15653   |           56773.5 |         4959 |           5.42 |
| edad_jefe   |    39 |    63   |    24   |               3   |              99   |           20 |           0.02 |
| tot_integ   |     2 |     4   |     2   |              -1   |               7   |         2040 |           2.23 |
| ocupados    |     1 |     2   |     1   |              -0.5 |               3.5 |         5149 |           5.63 |

## Cardinalidad de variables categóricas

- `ubica_geo`: 1112 categorías
- `tam_loc`: 4 categorías
- `est_socio`: 4 categorías
- `clase_hog`: 5 categorías
- `sexo_jefe`: 2 categorías
- `educa_jefe`: 11 categorías
