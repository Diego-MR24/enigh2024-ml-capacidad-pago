# Resumen de resultados — Notebook 03 (modelado)

## 1. Comparación de los 3 modelos (conjunto de prueba, n = 18,283)

| Umbral | Modelo | Accuracy | Recall clase 0 | Recall clase 1 | AUC |
|---|---|---|---|---|---|
| $15,000 (≈2 SM) | Regresión Logística | 0.717 | 0.731 | 0.708 | 0.800 |
| $15,000 (≈2 SM) | Random Forest | 0.703 | 0.595 | 0.769 | 0.762 |
| $15,000 (≈2 SM) | Red Neuronal (MLP) | 0.747 | 0.628 | 0.820 | 0.814 |
| Mediana (≈LPI urbana) | Regresión Logística | 0.720 | 0.722 | 0.718 | 0.798 |
| Mediana (≈LPI urbana) | Random Forest | 0.685 | 0.690 | 0.679 | 0.758 |
| Mediana (≈LPI urbana) | Red Neuronal (MLP) | 0.730 | 0.734 | 0.726 | 0.810 |

- La Red Neuronal (MLP) tiene el mejor AUC en ambos umbrales (0.814 y 0.810).
- Random Forest queda consistentemente en último lugar (AUC 0.762 y 0.758).
- Los resultados de validación cruzada (cv_auc_mean) son muy cercanos a los
  de test (diferencia ≤0.002 en todos los casos), lo que indica que no hay
  sobreajuste relevante.

## 2. Importancia de variables

- **Regresión Logística y Red Neuronal**: las variables más influyentes son
  consistentemente `clase_hog_3` (hogar ampliado), `clase_hog_2` (hogar
  nuclear), `educa_num` y `tasa_dependencia` (esta última con coeficiente
  negativo en Regresión Logística).
- **Random Forest**: `edad_jefe` domina con ~0.41 de importancia en ambos
  umbrales, muy por encima de cualquier otra variable — pese a que su
  correlación con el ingreso es prácticamente nula (r ≈ -0.01, según el
  EDA).
- Este patrón se repite igual en los dos umbrales ($15,000 y mediana),
  confirmando que no es un artefacto de un solo escenario.

## 3. Sensibilidad a `est_socio_num` (Red Neuronal)

| Umbral | Variante | Accuracy | Recall 0 | Recall 1 | AUC |
|---|---|---|---|---|---|
| $15,000 | Con est_socio_num | 0.747 | 0.628 | 0.820 | 0.814 |
| $15,000 | Sin est_socio_num | 0.747 | 0.639 | 0.814 | 0.813 |
| Mediana | Con est_socio_num | 0.730 | 0.734 | 0.726 | 0.810 |
| Mediana | Sin est_socio_num | 0.726 | 0.740 | 0.711 | 0.805 |

- Quitar `est_socio_num` apenas cambia el desempeño (diferencia de AUC
  ≤0.005 en ambos casos).
- En el umbral de $15,000 incluso sube levemente el recall de la clase 0
  (0.628 → 0.639) al quitar la variable.

## 4. Sensibilidad al balanceo de clases (umbral $15,000)

| Modelo | Variante | Accuracy | Recall 0 | Recall 1 | AUC |
|---|---|---|---|---|---|
| Regresión Logística | Sin balanceo | 0.734 | 0.571 | 0.834 | 0.800 |
| Regresión Logística | Con balanceo | 0.717 | 0.731 | 0.708 | 0.800 |
| Random Forest | Sin balanceo | 0.704 | 0.574 | 0.784 | 0.761 |
| Random Forest | Con balanceo | 0.703 | 0.595 | 0.769 | 0.762 |
| Red Neuronal (MLP) | Sin balanceo | 0.747 | 0.628 | 0.820 | 0.814 |
| Red Neuronal (MLP) | Con balanceo | 0.728 | 0.753 | 0.712 | 0.812 |

- En los 3 modelos, el balanceo sube el recall de la clase 0 entre ~2 y ~16
  puntos porcentuales, a cambio de bajar el recall de la clase 1 y la
  accuracy.
- El AUC se mantiene prácticamente igual en los 3 casos (diferencia
  ≤0.002), confirmando que el balanceo cambia el punto de operación, no la
  capacidad discriminativa del modelo.

## Conclusiones generales

1. El modelo más adecuado para ambos umbrales es la **Red Neuronal (MLP)**,
   seguida de cerca por la Regresión Logística (más interpretable). Random
   Forest necesitaría ajuste de hiperparámetros para ser competitivo.
2. La estructura del hogar (`clase_hog_2`, `clase_hog_3`) y el nivel
   educativo (`educa_num`) son los predictores más robustos en los modelos
   con mejor desempeño.
3. `est_socio_num` puede omitirse sin pérdida relevante de desempeño,
   reforzando la viabilidad de un estimador basado solo en variables
   demográficas básicas.
4. La elección de balancear o no las clases depende del caso de uso: sin
   balanceo se prioriza identificar hogares con capacidad de pago (crédito);
   con balanceo se prioriza identificar hogares vulnerables (política
   social), sin afectar el AUC.
