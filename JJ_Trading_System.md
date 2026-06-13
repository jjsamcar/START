# JJ Trading System

> Plataforma personal de análisis y ejecución de estrategias de trading, diseñada para crecer de forma modular e incremental.

---

## Tabla de Contenidos

1. [Visión General](#visión-general)
2. [Stack Tecnológico](#stack-tecnológico)
3. [Arquitectura de la Plataforma](#arquitectura-de-la-plataforma)
4. [Concepto Central: Instancias (Estrategia + Activo)](#concepto-central-instancias-estrategia--activo)
5. [Estructura de Carpetas](#estructura-de-carpetas)
6. [Módulos Planificados](#módulos-planificados)
7. [Estrategias](#estrategias)
8. [Roadmap](#roadmap)

---

## Visión General

**JJ Trading System** es un dashboard interactivo de trading construido en Python con Streamlit. El objetivo es centralizar el análisis de estrategias, la gestión de riesgo, la gestión de posiciones y el control del capital en una sola plataforma, escalable y fácil de mantener.

**Activos (universo en expansión):**
- `QQQ` — ETF Nasdaq 100 (datos CSV disponibles)
- `SPY` — ETF S&P 500 (próximamente)
- `IWM` — ETF Russell 2000 / small caps (planificado)
- **Large caps individuales** — múltiples acciones (futuro: AAPL, MSFT, NVDA, etc.)

> El módulo de datos es **agnóstico al activo**: se carga cualquier ticker sin hardcodear nada, para poder escalar a decenas de activos.

**Primera estrategia implementada:**
- Overnight Gap Trading

---

## Stack Tecnológico

| Herramienta | Rol | Notas |
|---|---|---|
| **Python** | Lenguaje principal | v3.10+ recomendado |
| **Streamlit** | Dashboard interactivo | UI sin necesidad de frontend separado |
| **Pandas** | Manipulación de datos | Lectura y procesamiento de CSV |
| **GitHub** | Control de versiones | Repositorio: `jjsamcar/START` |
| **Claude (AI)** | Asistente de desarrollo | Edición de código, ideas, documentación |
| **CSV local** | Fuente de datos | Datos históricos de precios (OHLCV) |

---

## Arquitectura de la Plataforma

La plataforma está diseñada con un principio central: **cada módulo es independiente y reemplazable**. Esto permite agregar nuevas estrategias o funcionalidades sin romper lo que ya existe.

```
JJ Trading System
│
├── 📊 Data Layer          → Carga y limpieza de datos (CSV, futuros: API)
├── 🧠 Strategy Layer      → Lógica de cada estrategia (una por módulo)
├── ⚙️  Config Layer        → Parámetros por instancia (estrategia + activo)
├── 📈 Analysis Layer      → Backtesting, métricas, visualizaciones
├── 💰 Risk & Capital      → Gestión de riesgo, sizing, money management
├── 📋 Position Manager    → Seguimiento de posiciones abiertas/cerradas
├── 🔗 Portfolio Layer     → Consolidación: drawdown unificado, correlación, capital total
└── 🖥️  Dashboard (UI)     → Streamlit — interfaz unificada para todo lo anterior
```

### Principios de diseño

- **Modularidad**: Cada estrategia vive en su propio archivo. Agregar una nueva no afecta las demás.
- **Lógica agnóstica a la UI**: La estrategia no sabe que existe un dashboard. El dashboard le pasa parámetros, la estrategia los usa. Así la misma estrategia corre desde un script, un notebook o el dashboard sin cambiar nada.
- **Escalabilidad**: La fuente de datos puede migrar de CSV local a una API (Yahoo Finance, Alpha Vantage, Polygon.io) sin cambiar la lógica de las estrategias.
- **Reproducibilidad**: Todo el código versionado en GitHub. Cualquier resultado es reproducible.
- **Iteración rápida**: Claude asiste en el desarrollo para implementar ideas rápidamente desde la conversación.

---

## Concepto Central: Instancias (Estrategia + Activo)

El corazón del diseño escalable. Una **estrategia** es lógica reutilizable; cada **activo** sobre el que se aplica genera una **instancia** independiente con su propia configuración.

> **No se duplica código, se duplica configuración.** Si mejoras la lógica base, todas las instancias se benefician automáticamente.

```
Overnight Gap (lógica base — 1 solo archivo)
│
├── instancia: OG_QQQ  → config propia (gap 0.5%–2%,  stop X, size Y)
├── instancia: OG_SPY  → config propia (gap 0.3%–1.5%, stop Z, size W)
└── instancia: OG_IWM  → config propia (gap 0.8%–3%,  stop …, size …)
        │
        └── cada instancia → genera trades → alimenta Riesgo y Posiciones
                                                    ↓
                                    🔗 Vista Consolidada del Portfolio
                              (drawdown unificado, correlación, capital total)
```

**Implicaciones de diseño:**

1. **Una lógica base por estrategia** — el código vive en un único archivo.
2. **Configuración independiente por activo** — cada instancia (estrategia+activo) tiene sus propios parámetros: rango de gap, stop, sizing, etc. Cada activo se comporta diferente, por eso se ajusta por separado.
3. **Riesgo y posiciones operan a nivel de instancia** — cada instancia tiene su propio stop, sizing, historial de trades y P&L.
4. **Capa de consolidación (Portfolio Layer)** — suma todas las instancias para dar drawdown unificado, correlación entre estrategias, capital total comprometido y performance combinada del "pool".

### Parámetros vs. Controles de UI

Un mismo parámetro vive en dos capas separadas:

| Elemento | Dónde vive | Ejemplo |
|---|---|---|
| **El parámetro** (el valor y su uso) | Strategy Layer / Config | `gap_min=0.5%`, `gap_max=2%` |
| **El control** (el slider para moverlo) | Dashboard Layer | Slider de rango de gap en la página de la estrategia |

```
Dashboard (slider gap 0.5%–2%)  →  parámetros  →  Strategy Layer (filtra señales)
```

---

## Estructura de Carpetas

```
JJ_Trading_System/
│
├── app.py                    # Entry point — Streamlit app principal
├── JJ_Trading_System.md      # Documento maestro (arquitectura, índice, roadmap)
│
├── data/
│   ├── raw/                  # CSV originales sin modificar
│   │   ├── QQQ.csv
│   │   ├── SPY.csv
│   │   └── IWM.csv
│   └── processed/            # Datos con indicadores calculados
│
├── strategies/
│   ├── __init__.py
│   ├── overnight_gap.py      # Estrategia #1: lógica base (1 solo archivo)
│   ├── configs/              # Configuración por instancia (estrategia + activo)
│   │   ├── overnight_gap_QQQ.yaml
│   │   ├── overnight_gap_SPY.yaml
│   │   └── overnight_gap_IWM.yaml
│   └── docs/                 # Documentación detallada por estrategia
│       └── 01_overnight_gap.md
│
├── analysis/
│   ├── backtester.py         # Motor de backtesting
│   └── metrics.py            # Sharpe, drawdown, win rate, etc.
│
├── risk/
│   ├── risk_manager.py       # Reglas de riesgo por instancia
│   ├── position_manager.py   # Gestión de posiciones
│   └── money_management.py   # Sizing, % de capital por trade
│
├── portfolio/
│   └── consolidator.py       # Drawdown unificado, correlación, capital total
│
├── dashboard/
│   ├── pages/
│   │   ├── 01_overview.py        # Resumen general del portafolio
│   │   ├── 02_strategies.py      # Vista por estrategia / instancia
│   │   ├── 03_risk.py            # Panel de riesgo
│   │   ├── 04_positions.py       # Posiciones abiertas y cerradas
│   │   └── 05_portfolio.py       # Vista consolidada del pool de estrategias
│   └── components/               # Widgets reutilizables
│
└── requirements.txt              # Dependencias del proyecto
```

---

## Módulos Planificados

### Implementados
- [ ] Carga de datos CSV (QQQ, SPY)
- [ ] Dashboard base en Streamlit

### En progreso / Próximos
- [ ] **Estrategia: Overnight Gap Trading**
- [ ] Backtesting básico con métricas clave
- [ ] Panel de visualización por estrategia

### Futuro
- [ ] **Sistema de instancias** — Estrategia base + config por activo (QQQ, SPY, IWM…)
- [ ] **Gestión de Riesgo** — Stop loss dinámico, riesgo máximo por operación, riesgo diario (por instancia)
- [ ] **Gestión de Posiciones** — Seguimiento de trades abiertos y cerrados, historial (por instancia)
- [ ] **Money Management** — Tamaño de posición basado en % de capital, Kelly Criterion
- [ ] **Portfolio Layer (consolidación)** — Drawdown unificado, correlación entre estrategias, capital total del pool
- [ ] Expansión de activos — IWM y large caps individuales
- [ ] Conexión a API de datos en tiempo real
- [ ] Alertas automáticas de señales
- [ ] Reporte de performance semanal/mensual

---

## Estrategias

> Cada estrategia tiene su propio documento detallado en `strategies/docs/`. Este maestro solo lista el índice.

| # | Estrategia | Activos | Estado | Documento |
|---|---|---|---|---|
| 1 | Overnight Gap Trading | QQQ, SPY, IWM | En definición | [`01_overnight_gap.md`](strategies/docs/01_overnight_gap.md) |

---

## Roadmap

```
Fase 1 — Fundamentos         ← Estamos aquí
  ✅ Definir arquitectura
  ⬜ Crear estructura de carpetas en repo
  ⬜ Cargar y visualizar datos QQQ

Fase 2 — Primera Estrategia
  ⬜ Definir reglas de Overnight Gap Trading
  ⬜ Implementar lógica en Python
  ⬜ Backtesting y métricas
  ⬜ Visualización en dashboard

Fase 3 — Instancias y Gestión de Capital
  ⬜ Sistema de instancias (estrategia + config por activo)
  ⬜ Módulo de gestión de riesgo (por instancia)
  ⬜ Módulo de gestión de posiciones (por instancia)
  ⬜ Money management integrado a estrategias

Fase 4 — Consolidación y Escala
  ⬜ Portfolio Layer: drawdown unificado y correlación
  ⬜ Expansión de activos (IWM, large caps)
  ⬜ Segunda estrategia
  ⬜ Datos en tiempo real
```

---

*Documento vivo — se actualiza conforme evoluciona el proyecto.*
