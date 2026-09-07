# ETL FIFA 21 · Arquitectura Medallion en Databricks

### Jorge Amat · David Plaza

## Objetivo del proyecto

Este proyecto es un ejercicio de la asignatura de ETL centrado en aplicar la **arquitectura Medallion (Bronze → Silver → Gold)** en **Databricks** usando **PySpark** y **Delta Lake**, sobre un dataset real y "sucio": [FIFA 21 messy raw dataset](https://www.kaggle.com/datasets/yagunnersya/fifa-21-messy-raw-dataset-for-cleaning-exploring).

Los objetivos de aprendizaje concretos son:

- **Spark / PySpark**: transformaciones sobre DataFrames distribuidos (`withColumn`, `regexp_extract`, `regexp_replace`, joins, agregaciones), y gestión de sesión en un entorno gestionado como Databricks.
- **Delta Lake**: lectura/escritura de tablas Delta, `saveAsTable`, `OPTIMIZE ... ZORDER`, `VACUUM`, y organización en catálogo/esquemas con Unity Catalog.
- **Transformaciones de datos reales**: normalización de tipos (alturas, pesos, variables monetarias), limpieza de texto, extracción de campos combinados, deduplicado, enriquecimiento con columnas derivadas.
- **Métricas de calidad del dato**: medir y comparar completitud, unicidad, validez, consistencia, exactitud y actualidad, antes y después de limpiar (Bronze vs. Silver).
- **Modelado analítico**: diseño de un esquema en estrella (dimensiones + tabla de hechos) para la capa Gold, orientado a consultas de explotación.

## Arquitectura

```
CSV (Kaggle, "messy") 
      │
      ▼
   Volume (almacenamiento en bruto, sin parsear)
      │
      ▼
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   BRONZE    │ --> │    SILVER    │ --> │    GOLD     │
│ tabla Delta │     │ tabla Delta  │     │ esquema     │
│ casi cruda  │     │ limpia +     │     │ en estrella │
│             │     │ data quality │     │ (dims+hechos)│
└─────────────┘     └──────────────┘     └─────────────┘
```

## Estructura del repositorio

```
.
├── README.md
├── 00_setup_catalogo.ipynb   # Cataálogo, esquemas, volumen y carga robusta del CSV
├── carga_delta.ipynb         # Landing → Bronze
├── transformaciones.ipynb    # Bronze → Silver (+ Data Quality Report)
├── MODELLING.ipynb           # Silver → Gold (esquema en estrella)

```

## Notebooks y orden de ejecución

| # | Notebook | Qué hace |
|---|----------|----------|
| 1 | `00_setup_catalogo.ipynb` | Crea el catálogo `fifa_catalog`, los esquemas `bronze`/`silver`/`gold`, un Volume para el CSV en bruto, lee el CSV con las opciones correctas (`multiLine`, `escape`) y escribe la tabla raw. |
| 2 | `carga_delta.ipynb` | Lee la tabla raw, normaliza nombres de columna y la guarda como tabla Delta en Bronze. |
| 3 | `transformaciones.ipynb` | Calcula el Data Quality Report sobre Bronze, aplica todas las transformaciones de limpieza/enriquecimiento, vuelve a calcular la calidad sobre el resultado y escribe Silver. |
| 4 | `MODELLING.ipynb` | Construye el esquema en estrella (`dim_player`, `dim_nationality`, `dim_club`, `dim_position`, `fact_player_stats`) y lo escribe en Gold. |

Hay que ejecutarlos en ese orden porque cada uno depende de la tabla que deja el anterior.




