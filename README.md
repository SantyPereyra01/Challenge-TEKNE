# Data & AI Challenge — Insurance Claims Fraud Detection

**Autor:** Santiago Pereyra  
**Challenge:** Tekne Data Labs  
**Fecha:** Abril 2026

---

## Contexto

Challenge técnico de Data & AI para Tekne Data Labs. Se trabaja con datos reales de siniestros de una aseguradora con el objetivo de demostrar capacidad de limpieza de datos, análisis de negocio y uso práctico de IA.

## Dataset

**Insurance Claims Fraud Data** — 3 tablas:

| Tabla | Registros | Descripción |
|-------|----------:|-------------|
| `insurance_data` | 10,000 | Transacciones de claims (tabla de hechos) |
| `employee_data` | 1,200 | Maestro de agentes/ajustadores |
| `vendor_data` | 600 | Maestro de vendors investigadores |

**Fuente:** `archive.zip` contiene los 3 CSVs originales.

## Estructura del repositorio

```
.
├── README.md
├── archive.zip                          # Dataset original (3 CSVs comprimidos)
│
├── 01_data_engineering_quality.ipynb     # Punto 1: Data Engineering & Quality (35%)
├── 02_analytics_negocio.ipynb           # Punto 2: Analytics de Negocio (35%)
├── 03_ia_aplicada_llm.ipynb             # Punto 3: IA Aplicada con LLM (20%)
├── 04_resumen_ejecutivo.docx            # Punto 4: Resumen Ejecutivo (10%)
│
├── data/                                # CSVs extraídos del ZIP (raw)
│   ├── insurance_data.csv
│   ├── employee_data.csv
│   └── vendor_data.csv
│
└── parquet/                             # Tablas normalizadas y limpias
    ├── insurance_data.parquet           # Tabla de hechos limpia
    ├── employee_data.parquet            # Maestro agentes limpio
    ├── vendor_data.parquet              # Maestro vendors limpio
    └── insurance_integrated.parquet     # Dataset integrado + features (input para puntos 2, 3 y 4)
```

## Notebooks

### 1. Data Engineering & Quality (35%)

**Archivo:** `01_Data_Engineering_&_Quality.ipynb`

**Qué cubre:**
- Extracción del ZIP y carga de los 3 CSVs
- Exploración detallada de cada tabla (tipos, nulls, duplicados, cardinalidad)
- Descripción del modelo de datos y validación de integridad referencial
- Documentación de 5 problemas de calidad encontrados y su resolución
- Normalización y limpieza de las 3 tablas
- Persistencia en formato Parquet
- Integración de las 3 tablas en dataset consolidado
- Cálculo de 6 features derivadas temporales
- Consultas SQL (DuckDB) contra las tablas Parquet

**Problemas de calidad documentados:**

| # | Problema | Resolución |
|---|----------|------------|
| 1 | Fechas como string | Casteo a `datetime64` |
| 2 | POSTAL_CODE sin leading zeros | `str` + `zfill(5)` |
| 3 | Nulls en categóricos | Imputación contextual |
| 4 | CLAIM_AMOUNT bucketeado (107 valores únicos) | Documentado como limitación |
| 5 | TENURE no correlaciona con antigüedad de póliza | Documentado + feature calculada |

**Features derivadas:**

| Feature | Cálculo |
|---------|---------|
| `DAYS_TO_REPORT` | `REPORT_DT - LOSS_DT` (días) |
| `POLICY_AGE_YEARS` | `LOSS_DT - POLICY_EFF_DT` (años) |
| `LOSS_RATIO` | `CLAIM_AMOUNT / PREMIUM_AMOUNT` |
| `AGENT_SENIORITY_YEARS` | `TXN_DATE_TIME - DATE_OF_JOINING` (años) |
| `OUT_OF_STATE_INCIDENT` | Flag: siniestro fuera del estado del cliente |
| `INCIDENT_TIME_CATEGORY` | Franja horaria del incidente |

### 2. Analytics de Negocio (35%)

**Archivo:** `02_analytics_negocio.ipynb`

**Qué cubre:**
- Definición y cálculo de 3 KPIs (tasa de rechazo, loss ratio, tiempo de reporte)
- Evolución mensual de volumen y monto de siniestros
- Identificación de 2 anomalías en los datos
- 2 insights accionables con dato de respaldo y acción recomendada
- Modelo predictivo de CLAIM_STATUS (Random Forest)

### 3. IA Aplicada con LLM (20%)

**Archivo:** `03_ia_aplicada_llm.ipynb`

**Qué cubre:**
- Caso de uso: Auditor Automático de Siniestros Sospechosos
- Diseño del prompt para auditoría de claims
- Demo funcional en Python con registros reales
- Explicación del business value

### 4. Resumen Ejecutivo (10%)

**Archivo:** `04_resumen_ejecutivo.docx`

**Qué cubre:**
- Resumen de hallazgos principales dirigido al director de siniestros
- Principales riesgos y alertas identificadas
- Recomendaciones de próximos pasos
- Tono ejecutivo, sin jerga técnica

## Stack técnico

| Herramienta | Uso |
|-------------|-----|
| Python 3.12 | Lenguaje principal |
| pandas | Manipulación de datos |
| numpy | Operaciones numéricas |
| matplotlib / seaborn | Visualizaciones |
| pyarrow | Lectura/escritura Parquet |
| DuckDB | Consultas SQL sobre Parquet |
| scikit-learn | Modelo predictivo |
| Anthropic API (Claude) | Caso de uso LLM |

## Cómo ejecutar

### Requisitos previos

```bash
pip install pandas numpy matplotlib seaborn pyarrow duckdb scikit-learn openpyxl
```

### Ejecución

Los notebooks se ejecutan en orden. El punto 1 genera los Parquets que consumen los puntos 2, 3 y 4.

```bash
# 1. Ejecutar punto 1 primero (genera los Parquets)
jupyter notebook 01_data_engineering_quality.ipynb

# 2. Luego los siguientes en cualquier orden
jupyter notebook 02_analytics_negocio.ipynb
jupyter notebook 03_ia_aplicada_llm.ipynb
```

El archivo `archive.zip` debe estar en el mismo directorio que los notebooks.

## Modelo de datos

```
┌─────────────────────┐        ┌──────────────────┐
│   employee_data     │        │   vendor_data     │
│   (maestro agentes) │        │   (maestro vendor)│
│                     │        │                   │
│   PK: AGENT_ID      │        │   PK: VENDOR_ID   │
│   1,200 registros   │        │   600 registros   │
└──────────┬──────────┘        └──────────┬────────┘
           │ 1                            │ 1
           │                              │
           │ N                            │ N (nullable)
┌──────────┴──────────────────────────────┴────────┐
│                insurance_data                     │
│              (tabla transaccional)                │
│                                                   │
│   PK: TRANSACTION_ID                              │
│   FK: AGENT_ID    → employee_data.AGENT_ID        │
│   FK: VENDOR_ID   → vendor_data.VENDOR_ID (NULL)  │
│   10,000 registros                                │
└───────────────────────────────────────────────────┘
```

## Hallazgos clave

- **Tasa de rechazo del 5%** — posible subdetección de fraude
- **Loss ratio promedio de 199x** — con extremos >1000x que no se filtran
- **15.6% de claims reportados el mismo día** — patrón asociado a preparación previa
- **32% de claims sin vendor investigador** — limita capacidad de detección
- **TENURE no correlaciona con antigüedad de póliza** — inconsistencia en el dato fuente

---

*Challenge diseñado por [Tekne Data Labs](https://www.teknedatalabs.com)*
