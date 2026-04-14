
README.md

Página
1
/
1
100 %
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

## Estructura del repositorio

```
.
├── README.md
├── .gitignore
├── .env                                     # API key (NO se sube a git)
├── archive.zip                              # Dataset original
│
├── 01_data_engineering_quality.ipynb         # Punto 1: Data Engineering & Quality (35%)
├── 02_analytics_negocio.ipynb               # Punto 2: Analytics de Negocio (35%)
├── 03_ia_aplicada_llm.ipynb                 # Punto 3: IA Aplicada con LLM (20%)
├── 04_resumen_ejecutivo.docx                # Punto 4: Resumen Ejecutivo (10%)
│
├── data/
│   ├── bronze/                              # CSVs raw extraídos del ZIP
│   ├── silver/                              # Parquet limpios por tabla
│   └── gold/                                # Dataset integrado + features
│
├── db/
│   └── tekne_claims.duckdb                  # Base SQL local
│
└── outputs_llm/                             # Outputs del auditor LLM
```

## Notebooks

### 1. Data Engineering & Quality (35%)
- Extracción ZIP → exploración → modelo de datos con diagrama.
- 5 problemas de calidad documentados y resueltos.
- Normalización con funciones reutilizables (snake_case, tipado, padding).
- Persistencia Parquet (bronze → silver → gold) + DuckDB.
- 7 features derivadas temporales y analíticas.

### 2. Analytics de Negocio (35%)
- 3 KPIs: tasa de rechazo, loss ratio, tiempo de reporte.
- Evolución mensual global y por tipo de seguro.
- 3 anomalías + 1 hallazgo complementario.
- 2 insights accionables con dato y acción concreta.
- Benchmark predictivo: LogReg + Random Forest + feature importance.

### 3. IA Aplicada con LLM (20%)
- Caso de uso: Auditor Automático de Siniestros Sospechosos.
- Score heurístico ponderado (8 flags).
- Prompt con 5 secciones + demo con OpenAI (gpt-4o-mini).
- Fallback offline + estimación de costos.

### 4. Resumen Ejecutivo (10%)
- Documento Word dirigido al Director de Siniestros.
- KPIs, riesgos, 5 recomendaciones. Tono ejecutivo, sin jerga técnica.

## Stack técnico

| Herramienta | Uso |
|-------------|-----|
| Python 3.13 | Lenguaje principal |
| pandas / numpy | Manipulación de datos |
| pyarrow / DuckDB | Parquet + SQL |
| matplotlib / seaborn | Visualizaciones |
| scikit-learn | Modelos predictivos |
| OpenAI API | Auditor LLM (gpt-4o-mini) |

## Cómo ejecutar

```bash
pip install pandas numpy pyarrow duckdb matplotlib seaborn scikit-learn openai python-dotenv
```

Crear `.env` con: `OPENAI_API_KEY=sk-proj-tu-key` (solo para Punto 3).

Los notebooks se ejecutan en orden (Punto 1 genera los Parquets para los demás).

## Hallazgos clave

- **Tasa de rechazo del 5%** — posible subdetección de fraude.
- **Life Insurance con loss ratio 728x** — 3,5x el promedio, con la menor tasa de rechazo.
- **Claims con múltiples red flags no se rechazan más** — sin triaje automatizado.
- **Segmentación de riesgo invertida** — H tiene menor rechazo que L.
- **32% de claims sin vendor** — limita capacidad de investigación.

---

*Challenge diseñado por [Tekne Data Labs](https://www.teknedatalabs.com)*
Mostrando README.md.
