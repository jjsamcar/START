# JJ Trading System

> Plataforma personal de análisis y ejecución de estrategias de trading, diseñada para crecer de forma modular e incremental.

---

## Tabla de Contenidos

1. [Visión General](#visión-general)
2. [Stack Tecnológico](#stack-tecnológico)
3. [Arquitectura de la Plataforma](#arquitectura-de-la-plataforma)
4. [Estructura de Carpetas](#estructura-de-carpetas)
5. [Módulos Planificados](#módulos-planificados)
6. [Estrategias](#estrategias)
7. [Roadmap](#roadmap)

---

## Visión General

**JJ Trading System** es un dashboard interactivo de trading construido en Python con Streamlit. El objetivo es centralizar el análisis de estrategias, la gestión de riesgo, la gestión de posiciones y el control del capital en una sola plataforma, escalable y fácil de mantener.

**Activos iniciales:**
- `QQQ` — ETF Nasdaq 100 (datos CSV disponibles)
- `SPY` — ETF S&P 500 (próximamente)

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
├── 📈 Analysis Layer      → Backtesting, métricas, visualizaciones
├── 💰 Risk & Capital      → Gestión de riesgo, sizing, money management
├── 📋 Position Manager    → Seguimiento de posiciones abiertas/cerradas
└── 🖥️  Dashboard (UI)     → Streamlit — interfaz unificada para todo lo anterior
```

### Principios de diseño

- **Modularidad**: Cada estrategia vive en su propio archivo. Agregar una nueva no afecta las demás.
- **Escalabilidad**: La fuente de datos puede migrar de CSV local a una API (Yahoo Finance, Alpha Vantage, Polygon.io) sin cambiar la lógica de las estrategias.
- **Reproducibilidad**: Todo el código versionado en GitHub. Cualquier resultado es reproducible.
- **Iteración rápida**: Claude asiste en el desarrollo para implementar ideas rápidamente desde la conversación.

---

## Estructura de Carpetas

```
JJ_Trading_System/
│
├── app.py                    # Entry point — Streamlit app principal
├── JJ_Trading_System.md      # Este documento
│
├── data/
│   ├── raw/                  # CSV originales sin modificar
│   │   ├── QQQ.csv
│   │   └── SPY.csv
│   └── processed/            # Datos con indicadores calculados
│
├── strategies/
│   ├── __init__.py
│   └── overnight_gap.py      # Estrategia #1: Overnight Gap Trading
│
├── analysis/
│   ├── backtester.py         # Motor de backtesting
│   └── metrics.py            # Sharpe, drawdown, win rate, etc.
│
├── risk/
│   ├── risk_manager.py       # Reglas de riesgo por operación
│   ├── position_manager.py   # Gestión de posiciones
│   └── money_management.py   # Sizing, % de capital por trade
│
├── dashboard/
│   ├── pages/
│   │   ├── 01_overview.py        # Resumen general del portafolio
│   │   ├── 02_strategies.py      # Vista por estrategia
│   │   ├── 03_risk.py            # Panel de riesgo
│   │   └── 04_positions.py       # Posiciones abiertas y cerradas
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
- [ ] **Gestión de Riesgo** — Stop loss dinámico, riesgo máximo por operación, riesgo diario
- [ ] **Gestión de Posiciones** — Seguimiento de trades abiertos y cerrados, historial
- [ ] **Money Management** — Tamaño de posición basado en % de capital, Kelly Criterion
- [ ] Conexión a API de datos en tiempo real
- [ ] Alertas automáticas de señales
- [ ] Reporte de performance semanal/mensual

---

## Estrategias

### 1. Overnight Gap Trading
> **Estado**: En definición

| Campo | Detalle |
|---|---|
| **Activos** | QQQ, SPY |
| **Timeframe** | Diario |
| **Tipo** | Gap al cierre/apertura |
| **Descripción** | Por definir conforme avance la lectura |
| **Parámetros** | Por definir |
| **Reglas de entrada** | Por definir |
| **Reglas de salida** | Por definir |
| **Gestión de riesgo** | Por definir |

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

Fase 3 — Gestión de Capital
  ⬜ Módulo de gestión de riesgo
  ⬜ Módulo de gestión de posiciones
  ⬜ Money management integrado a estrategias

Fase 4 — Escala
  ⬜ Segunda estrategia
  ⬜ Múltiples activos
  ⬜ Datos en tiempo real
```

---

*Documento vivo — se actualiza conforme evoluciona el proyecto.*
