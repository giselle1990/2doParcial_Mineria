# Segundo Parcial — Minería de Datos II

## Cloud Provider Analytics: Pipeline Lambda con PySpark + Astra DB

Este repositorio contiene el desarrollo del **MVP técnico del Segundo Parcial de Minería de Datos II**. El trabajo implementa un pipeline de datos para analítica de consumo cloud, utilizando **PySpark**, arquitectura **Lambda**, arquitectura medallón y carga final en **Astra DB / Cassandra**.

---

## Sitio web dinámico del parcial

El parcial también se presenta en una página web dinámica, donde se explica el proyecto de forma visual y ordenada punto por punto.

**Link a la web:**

[Ver presentación dinámica del parcial](https://giselle1990.github.io/2doParcial_Mineria/entrega_cloud_provider_analytics.html)

La web incluye:

- Objetivo del MVP.
- Arquitectura general del pipeline.
- Explicación de cada sección del código.
- Flujo Landing → Bronze → Silver → Gold → Serving.
- Reglas de calidad y quarantine.
- Modelo Gold FinOps.
- Serving en Astra DB.
- Validación de idempotencia.
- Resultados ejecutivos y gráficos.

---

## Archivos del repositorio

| Archivo | Descripción |
|---|---|
| `2doParcial.ipynb` | Notebook principal desarrollado en Google Colab. Contiene el código completo del MVP técnico. |
| `entrega_cloud_provider_analytics.html` | Página web dinámica para visualizar y recorrer el parcial de forma interactiva. |
| `README.md` | Descripción del proyecto, instrucciones de uso y enlaces principales. |

---

## Descripción técnica del proyecto

El pipeline implementado procesa datos de consumo cloud mediante dos caminos de ingesta:

1. **Batch**: procesa archivos maestros en formato CSV.
2. **Streaming**: procesa eventos de uso cloud en formato JSONL mediante Structured Streaming.

Luego, los datos avanzan por las capas de una arquitectura medallón:

```text
Landing → Bronze → Silver → Gold → Serving
```

### Landing

Capa de entrada con archivos crudos:

- `customers_orgs.csv`
- `users.csv`
- `billing_monthly.csv`
- `usage_events_stream/*.jsonl`

### Bronze

Capa de ingesta inicial. Se aplican:

- Esquemas explícitos.
- Metadata técnica.
- Deduplicación.
- Particionado.
- Checkpoints para streaming.

### Silver

Capa de limpieza y calidad. Se aplican:

- Normalización de campos.
- Enriquecimiento con datos maestros.
- Reglas de calidad.
- Separación de registros inválidos en quarantine.

### Gold

Capa analítica final. Se genera el mart:

```text
org_daily_usage_by_service
```

El grano de análisis es:

```text
organización + fecha + servicio
```

Métricas principales:

- Costo diario en USD.
- Cantidad de requests.
- Valor total procesado.
- CPU hours.
- Storage GB hours.
- Tokens GenAI.
- Huella de carbono.
- Flags de anomalía.

### Serving

La capa final publica los datos en **Astra DB**, utilizando un modelo orientado a consultas.

Tabla principal:

```sql
CREATE TABLE IF NOT EXISTS default_keyspace.org_daily_usage_by_service (
  org_id TEXT,
  usage_date DATE,
  service TEXT,
  daily_cost_usd DOUBLE,
  requests BIGINT,
  total_value DOUBLE,
  cpu_hours DOUBLE,
  storage_gb_hours DOUBLE,
  genai_tokens DOUBLE,
  carbon_kg DOUBLE,
  anomaly_flag TEXT,
  PRIMARY KEY ((org_id, usage_date), service)
);
```

---

## Cómo usar el proyecto

### 1. Clonar el repositorio

```bash
git clone https://github.com/giselle1990/2doParcial_Mineria.git
```

### 2. Ingresar a la carpeta del proyecto

```bash
cd 2doParcial_Mineria
```

### 3. Abrir el notebook en Google Colab

El archivo principal es:

```text
2doParcial.ipynb
```

Puede abrirse desde GitHub o descargarse y subirse manualmente a Google Colab.

### 4. Ejecutar el notebook

El notebook contiene las secciones necesarias para:

- Instalar dependencias.
- Inicializar Spark.
- Crear rutas del datalake.
- Procesar archivos batch.
- Procesar eventos streaming.
- Generar las capas Bronze, Silver y Gold.
- Cargar resultados en Astra DB.
- Ejecutar consultas.
- Validar idempotencia.
- Mostrar métricas finales.

### 5. Ver la web dinámica

La página web puede abrirse desde GitHub Pages:

[Ver presentación dinámica](https://giselle1990.github.io/2doParcial_Mineria/entrega_cloud_provider_analytics.html)

También puede abrirse localmente descargando el archivo:

```text
entrega_cloud_provider_analytics.html
```

---

## Requisitos técnicos

El notebook utiliza:

- Python.
- PySpark 3.5.0.
- OpenJDK 11.
- Spark Structured Streaming.
- Parquet.
- Astra DB / Cassandra.
- `spark-cassandra-connector`.
- `astrapy` como alternativa de carga mediante Data API.

---

## Validaciones implementadas

El proyecto incluye validaciones de:

- Esquemas explícitos.
- Deduplicación en batch.
- Deduplicación en streaming.
- Reglas de calidad en Silver.
- Quarantine de registros inválidos.
- Agregaciones Gold.
- Carga en Astra DB.
- Consultas sobre la capa serving.
- Idempotencia en re-ejecución.

---

## Resultado final

El resultado del trabajo es un pipeline funcional que permite pasar desde datos crudos hasta métricas listas para consulta y análisis ejecutivo.

La solución integra procesamiento batch, streaming, control de calidad, modelado analítico y serving en base NoSQL, mostrando un flujo completo de ingeniería de datos aplicado a un caso de analítica de consumo cloud.
