# SAA: Sistemas de Aprendizaje Automático

Repositorio de entregas del curso SI7002 Sistemas de Aprendizaje Automático, Maestría en Ciencia de Datos y Analítica, Universidad EAFIT.

Autor: Andrés Felipe Velasco Hernández

## Laboratorio N2: transformación de datos, de Raw a Features

Caso BookStore Online. El laboratorio construye las capas Curated y Features del data lake en AWS, con Glue y Step Functions, a partir de la zona Raw que dejó el Laboratorio N1.

### Contenido

```
data-platform/
  orchestration/
    step-functions/
      bookstore_curated.asl.json
      bookstore_features.asl.json
      bookstore_churn_features.asl.json
```

| Archivo | Máquina de estados | Qué hace |
|---|---|---|
| bookstore_curated.asl.json | bookstore-curated | Ejecuta en secuencia los jobs de Curated transaccional y behavioral |
| bookstore_features.asl.json | bookstore-features | Churn en modo scoring y, al terminar, recomendación, para un mismo corte |
| bookstore_churn_features.asl.json | bookstore-churn-features | Solo churn, con dataset_mode en la entrada; genera los snapshots históricos de training, validación y test |

### Jobs de Glue

Glue 5.1, workers G.1X, 10 workers, concurrencia máxima 1. Se crearon con AWS CLI.

- glue_curated_transactional_bookstore
- glue_curated_behavioral_bookstore
- glue_features_churn_bookstore
- glue_features_recommendation_bookstore

### Ejecuciones realizadas

| Máquina de estados | Ejecución | Entrada |
|---|---|---|
| bookstore-curated | curated-20261006-001 | {} |
| bookstore-features | churn-scoring-20260901-v2 | scoring, corte 2026-09-01, lookback_days 180 |
| bookstore-churn-features | churn-training-20250901-v2 | training, corte 2025-09-01 |
| bookstore-churn-features | churn-validation-20260501-v2 | training, corte 2026-05-01 |
| bookstore-churn-features | churn-test-20260701-v2 | training, corte 2026-07-01 |

Entrada de los cortes históricos:

```json
{
  "dataset_mode": "training",
  "cutoff_timestamp": "2025-09-01T00:00:00Z",
  "data_as_of_timestamp": "2026-09-01T23:59:51Z"
}
```

### Cómo recrear una máquina de estados

```
aws stepfunctions create-state-machine \
  --name bookstore-curated \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/LabRole \
  --type STANDARD \
  --definition file://data-platform/orchestration/step-functions/bookstore_curated.asl.json
```

Las otras dos se crean igual, cambiando el nombre y el archivo.

### Notas

- El nombre de cada ejecución se usa como run_id y relaciona los datos con su manifiesto y su reporte de calidad.
- No se incluyen los archivos generados de dist ni el código de los adaptadores de Glue, que pertenecen al repositorio del curso (SI7002/ml-platform).
