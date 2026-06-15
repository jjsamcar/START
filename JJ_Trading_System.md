# JJ Trading System

> Plataforma personal de análisis y ejecución de estrategias de trading, diseñada para crecer de forma modular e incremental.

---

## Tabla de Contenidos

1. [Visión General](#visión-general)
2. [Stack Tecnológico](#stack-tecnológico)
3. [Arquitectura de la Plataforma](#arquitectura-de-la-plataforma)
4. [Concepto Central: Instancias (Estrategia + Activo)](#concepto-central-instancias-estrategia--activo)
5. [El Pool: Registro y Consolidación de Instancias](#el-pool-registro-y-consolidación-de-instancias)
6. [Análisis Cuantitativo](#análisis-cuantitativo)
7. [Filtros](#filtros)
8. [Organización del Dashboard](#organización-del-dashboard)
9. [Persistencia y Rendimiento](#persistencia-y-rendimiento)
10. [Estructura de Carpetas](#estructura-de-carpetas)
11. [Módulos Planificados](#módulos-planificados)
12. [Estrategias](#estrategias)
13. [Roadmap](#roadmap)

---

## Visión General

**JJ Trading System** es un dashboard interactivo de trading construido en Python con Streamlit. El objetivo es centralizar el análisis de estrategias, la gestión de riesgo, la gestión de posiciones y el control del capital en una sola plataforma, escalable y fácil de mantener.

**Activos:**
- `QQQ` — ETF Nasdaq 100 (datos disponibles)
- `SPY` — ETF S&P 500 (próximamente)
- `IWM` — ETF Russell 2000 / small caps (planificado)
- Large caps individuales (planificado)

> El módulo de datos es **agnóstico al activo**: se carga cualquier ticker por su nombre, sin hardcodear nada.

**Primera estrategia:**
- Overnight Gap Trading

---

## Stack Tecnológico

| Herramienta | Rol | Notas |
|---|---|---|
| **Python** | Lenguaje principal | v3.10+ recomendado |
| **Streamlit** | Dashboard interactivo | UI sin necesidad de frontend separado |
| **Pandas** | Manipulación de datos | Lectura y procesamiento de CSV |
| **GitHub** | Control de versiones y fuente de datos | Repositorio: `jjsamcar/START` |
| **Streamlit Cloud** | Hosting del dashboard | Lee código y datos desde el repo |
| **CSV** | Fuente de datos | Históricos de precios (OHLCV) en `data/raw/` |

---

## Arquitectura de la Plataforma

La plataforma está diseñada con un principio central: **cada módulo es independiente y reemplazable**. Esto permite agregar nuevas estrategias o funcionalidades sin romper lo que ya existe.

```
JJ Trading System
│
├── 📊 Data Layer          → Carga y limpieza de datos (CSV)
├── 🧠 Strategy Layer      → Lógica de cada estrategia (una por módulo)
├── ⚙️  Config Layer        → Parámetros por instancia (estrategia + activo)
├── 📈 Analysis Layer      → Motor genérico: backtest, métricas, Montecarlo (se aplica por instancia)
├── 💰 Risk & Capital      → Gestión de riesgo, sizing, money management
├── 📋 Position Manager    → Seguimiento de posiciones abiertas/cerradas
├── 🔗 Portfolio Layer     → Consolidación: drawdown unificado, correlación, capital total
└── 🖥️  Dashboard (UI)     → Streamlit — interfaz unificada para todo lo anterior
```

### Principios de diseño

- **Modularidad**: Cada estrategia vive en su propio archivo. Agregar una nueva no afecta las demás.
- **Lógica agnóstica a la UI**: La estrategia no sabe que existe un dashboard. El dashboard le pasa parámetros, la estrategia los usa.
- **Datos agnósticos al activo**: La carga de datos funciona con cualquier ticker, sin hardcodear.
- **Reproducibilidad**: Todo el código y los datos versionados en GitHub.

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

## El Pool: Registro y Consolidación de Instancias

El **pool** es el conjunto de instancias activas que forman el portfolio. La clave del diseño: las instancias se **declaran**, no se programan a mano. Como la lógica base es única y solo cambia la config por activo, incluir una instancia en el pool se reduce a registrarla.

### Flujo para agregar una instancia al pool

```
1. Creas el archivo de config        →  strategies/configs/overnight_gap_SPY.yaml
2. La registras en el pool           →  portfolio/pool.yaml (lista de instancias activas)
3. El loader la carga automáticamente
```

> El código base **nunca se toca**. Solo agregas configs y las activas. Si mejoras `overnight_gap.py`, todas las instancias mejoran a la vez.

### El registro: `portfolio/pool.yaml`

```yaml
# Lista de instancias activas en el portfolio
instances:
  - id: OG_QQQ
    strategy: overnight_gap          # apunta a la lógica base
    config: overnight_gap_QQQ.yaml   # apunta a sus parámetros
    enabled: true
    capital_pct: 40                  # % del capital total asignado

  - id: OG_SPY
    strategy: overnight_gap          # MISMO código base
    config: overnight_gap_SPY.yaml   # OTRO activo, otra config
    enabled: true
    capital_pct: 35

  - id: OG_IWM
    strategy: overnight_gap
    config: overnight_gap_IWM.yaml
    enabled: false                   # configurada pero apagada (fuera del pool)
```

### Qué hace el loader por debajo

```
para cada instancia activa (enabled: true):
    cargar lógica base   (overnight_gap.py)
    + cargar su config   (overnight_gap_SPY.yaml)
    + cargar datos       (SPY.csv)
    → genera trades de esa instancia
    → alimenta Riesgo y Posiciones

luego, la Portfolio Layer suma TODAS las instancias activas
    → drawdown unificado, correlación, capital total
```

### Ventajas

- **Activar/desactivar** una instancia = cambiar `enabled: true/false` (sin borrar nada).
- **Asignación de capital** centralizada en el pool (`capital_pct`).
- **Mismo código base, distintos activos**: `OG_QQQ` y `OG_SPY` apuntan al mismo `overnight_gap.py`; solo cambian config y CSV.
- En el **dashboard**, el pool se vuelve una tabla con checkboxes para encender/apagar instancias y ver el efecto en el drawdown unificado en tiempo real.

---

## Análisis Cuantitativo

El módulo `analysis/` es un **motor genérico**: el código de backtest y métricas se escribe una sola vez y se **aplica por instancia**. El resultado es por estrategia+activo; el código es compartido (no se duplica por cada una).

```
OG_QQQ trades ─┐
OG_SPY trades ─┼─→  analysis/ (motor único)  ─→  métricas por instancia
OG_IWM trades ─┘                                        ↓
                                          portfolio/ consolida todas
```

### Métricas estándar (set ampliado)

Todas las instancias se evalúan con el mismo set de métricas:

| Grupo | Métricas |
|---|---|
| Rentabilidad | Return total, CAGR |
| Riesgo ajustado | Sharpe, Sortino, Calmar |
| Drawdown | Max DD, duración del Max DD, recovery factor |
| Operativa | Win %, Profit Factor, Expectancy, # trades, racha máxima de pérdidas |
| Eficiencia | Exposure (% de tiempo en mercado) |
| Comparativa | vs. Buy & Hold del mismo activo |

### Costos y supuestos de ejecución (por instancia)

El backtester resta costos para dar resultados realistas. Se modelan al estilo Interactive Brokers (comisión por acción) y son configurables por instancia.

```yaml
# en overnight_gap_QQQ.yaml
execution:
  commission_per_share: 0.0035   # IBKR Pro tiered (~$0.0035/acción)
  min_commission: 0.35           # mínimo por orden
  slippage_pct: 0.0001           # ~0.01% en ETFs líquidos
  entry_price: open              # open / close / next_open
  exit_price: close              # define a qué precio entra/sale el backtest
```

### Benchmark: Buy & Hold

Cada instancia muestra sus métricas junto a las de comprar y mantener el mismo activo, para saber si la estrategia aporta valor real frente a no hacer nada.

> **Supuesto de datos**: los precios están ajustados **solo por splits** (export de TradingView por defecto), no por dividendos. El benchmark B&H queda ligeramente subestimado (no incluye ~0.6% de QQQ / ~1.3% de SPY anual en dividendos). Tenerlo en cuenta al comparar.

### Optimización de parámetros (In-Sample)

El ajuste de parámetros (ej. rango de gap) se hace sobre el período IS. Se busca **robustez**, no el punto óptimo frágil: zonas donde los valores vecinos también funcionan.

- **Tab separada "Optimización"** en la página *Strategies*: corre un grid de combinaciones y muestra un **heatmap de robustez** (ej. Sharpe en función de `gap_min` × `gap_max`). Vive ahí para no saturar la vista principal.
- **Decisión final en el tab principal**: tras analizar el heatmap, eliges la zona robusta y ajustas el slider en la vista principal.

### In-Sample / Out-of-Sample (por instancia)

Para hacer cada instancia más robusta, su backtest se divide en períodos de fechas. El rango IS/OOS es **por instancia** (cada activo se afina por separado) y vive en su config `.yaml`.

- **In-Sample (IS)**: período donde se ajustan/optimizan los parámetros.
- **Out-of-Sample (OOS)**: período reservado para validar que la estrategia no está sobreajustada.
- **IS y OOS son listas de bloques**: cada uno puede tener varios rangos de fecha, no uno solo. Permite cubrir distintos regímenes de mercado (alcista, bajista, crisis) e intercalar IS/OOS en el tiempo en vez de partir la historia en dos mitades.
- Cada bloque se puede **activar/desactivar** (`enabled`), y hay exclusiones globales que se ignoran en cualquier período.

```yaml
# en overnight_gap_QQQ.yaml
periods:
  in_sample:                                # lista de bloques
    - { start: 2015-01-01, end: 2018-12-31, enabled: true }
    - { start: 2020-06-01, end: 2022-12-31, enabled: true }
  out_sample:                               # lista de bloques
    - { start: 2019-01-01, end: 2020-05-31, enabled: true }
    - { start: 2023-01-01, end: 2025-12-31, enabled: true }
  exclude:                                  # tramos a ignorar de cualquier período
    - { start: 2020-02-15, end: 2020-04-15, enabled: true }   # ej. crash COVID
```

El backtester une los bloques activos de cada conjunto y calcula las métricas sobre ellos. En el dashboard, esto se controla con rangos de fecha y los resultados se ven por separado (Completo / Solo IS / Solo OOS).

### Análisis de Montecarlo (por instancia)

Vive en `analysis/montecarlo.py`. Toma los trades ya calculados de una instancia y corre N simulaciones reordenando/remuestreando para estimar la robustez (distribución de returns, peor drawdown esperado, probabilidad de ruina). Se aplica por instancia, igual que el backtester.

### Correlaciones (a nivel de portfolio)

Comparan el comportamiento entre instancias, así que viven en `portfolio/consolidator.py` (no en una instancia individual). Resultado: matriz de correlación para saber qué está y qué no está correlacionado dentro del pool.

---

## Filtros

Los filtros permiten incluir o excluir trades según condiciones (eventos, día de la semana, horario). **La decisión de qué filtrar es por instancia** (vive en su config); los **datos compartidos** (fechas de eventos) viven en un archivo único.

| Cosa | Alcance | Dónde vive |
|---|---|---|
| Datos (fechas de eventos) | Compartido | `data/events.csv` (uno solo) |
| Decisión (qué filtros aplicar) | Por instancia | sección `filters` del `.yaml` |

### Formato estándar de eventos: `data/events.csv`

Un solo archivo con columnas fijas para todos los tipos de evento.

```csv
date,type,asset,name
2024-01-31,fed,ALL,FOMC Decision
2024-02-13,macro,ALL,CPI
2024-03-01,macro,ALL,NFP
2024-01-25,earnings,QQQ,Big tech earnings week
2024-02-19,holiday,ALL,Presidents Day
```

| Columna | Qué es | Valores |
|---|---|---|
| `date` | Fecha del evento | `YYYY-MM-DD` (ISO) |
| `type` | Tipo de evento | `fed`, `macro`, `earnings`, `holiday` |
| `asset` | A qué activo aplica | un ticker (`QQQ`) o `ALL` (mercado) |
| `name` | Etiqueta legible | texto libre, opcional |

**Reglas de consistencia:** fechas siempre `YYYY-MM-DD`; `type` en minúsculas de la lista permitida; eventos de mercado usan `asset=ALL`; sin filas duplicadas (`date`+`type`+`asset`).

### Filtros por instancia (familia `filters` en config)

Todos los filtros se agrupan bajo `filters` en el `.yaml` de la instancia. Agregar uno nuevo = una entrada más, sin rediseñar.

```yaml
# en overnight_gap_QQQ.yaml
filters:
  events:                          # usa data/events.csv
    exclude_fed: true
    exclude_earnings: true
    exclude_macro: false
    offset_days: 0                 # 0 = solo el día; ±1 = día previo/siguiente
  weekday:
    exclude: [monday]              # no necesita datos
  time:
    session: regular               # regular / extended (relevante en intradía)
```

### En el dashboard

En la página *Strategies*, cada filtro es un check o selector, y se muestra la **comparación con/sin** para decidir si el filtro mejora la estrategia.

```
FILTROS DE EVENTOS (OG_QQQ)
  ☑ Excluir días Fed        ☑ Excluir earnings     ☐ Excluir macro

           Todos   Sin Fed   Sin earnings
  Sharpe   1.45     1.61       1.52
  MaxDD   -12%     -9.5%      -10.2%
```

El motor `analysis/` lee la sección `filters`, filtra los trades antes de calcular métricas, y aplica los filtros de forma uniforme a cualquier instancia.

---

## Organización del Dashboard

El dashboard se organiza en páginas (menú lateral de Streamlit). El **análisis por instancia** vive en una sola página con selectores; el **consolidado** en otra.

| Página | Alcance | Contenido |
|---|---|---|
| **Overview** | Global | Resumen general del portafolio |
| **Strategies** | Por instancia | **Tab principal**: selectores estrategia+activo · sliders · backtest IS/OOS · métricas (vs B&H) · equity curve · tabla de trades · Montecarlo · filtros · Exportar configuración. **Tab Optimización**: grid + heatmap de robustez |
| **Risk** | Por instancia | Stop, riesgo máximo, sizing |
| **Positions** | Por instancia | Trades abiertos y cerrados |
| **Portfolio** | Consolidado | Pool (on/off), drawdown unificado, **matriz de correlaciones**, capital total |

> Una sola página *Strategies* sirve para analizar **cualquier** instancia: cambias los selectores de arriba y el motor `analysis/` recalcula los resultados de la instancia elegida. No hace falta una página por estrategia+activo.

---

## Persistencia y Rendimiento

### Entorno de ejecución

| Aspecto | Decisión |
|---|---|
| **Dónde corre** | Streamlit Community Cloud (no local) |
| **Disco de Streamlit Cloud** | Efímero — se borra en cada reinicio/sleep. No se puede usar para persistir. |
| **Datos de activos (CSV)** | Viven en el repo (`data/raw/*.csv`) para que Streamlit Cloud los lea. |
| **Escritura desde el dashboard** | ❌ No persiste. El dashboard solo lee, nunca guarda. |

### Configuraciones: dónde viven y cómo se guardan

Las configs por activo (`strategies/configs/<estrategia>_<TICKER>.yaml`) viven en **GitHub**, no en Streamlit Cloud (disco efímero, no persiste). Hay dos formas de cambiar parámetros:

| Forma | Dónde | ¿Persiste? |
|---|---|---|
| **Mover sliders en el dashboard** | Sesión actual (memoria) | ❌ Temporal — para experimentar |
| **Guardar el `.yaml` en GitHub** | Repo | ✅ Permanente |

**Flujo de guardado (botón "Exportar configuración"):** el dashboard tiene un botón que genera el texto YAML con los valores actuales de los sliders. El usuario lo copia y lo commitea al archivo de config en GitHub. La próxima vez que la app arranca, lee esos valores.

```
mueves sliders → [Exportar configuración] → copias el YAML → commit a GitHub → queda fijo
```

### Principio: separar "calcular" de "ver"

> **El dashboard NO corre la estrategia. Solo lee y muestra.** La lógica recibe datos + parámetros, calcula trades y métricas. El dashboard consume el resultado.

### Cálculo: recalcular al cargar con caché

El dashboard recalcula los trades al arrancar y cachea el resultado en memoria con `@st.cache_data`. Los datos son diarios y de pocos activos, así que el cálculo es de menos de un segundo por instancia. No se persisten resultados.

```
Streamlit Cloud arranca
   → lee CSVs desde el repo (data/raw/*.csv)
   → loader recalcula los trades de cada instancia
   → @st.cache_data guarda el resultado en memoria
   → mientras la app esté despierta, no recalcula
```

```python
@st.cache_data
def load_instance_results(instance_id):
    # lee CSV + config y calcula; el resultado queda cacheado en memoria
    return run_instance(instance_id)
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
│   ├── raw/                  # CSV de precios (OHLCV), una por activo
│   │   ├── QQQ.csv
│   │   ├── SPY.csv
│   │   └── IWM.csv
│   └── events.csv           # Fechas de eventos (fed, macro, earnings, holiday)
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
│   ├── backtester.py         # Motor de backtest (IS/OOS, costos, ejecución, B&H)
│   ├── metrics.py            # Set ampliado: Sharpe, Sortino, Calmar, PF, etc.
│   ├── montecarlo.py         # Simulaciones de robustez por instancia
│   └── optimizer.py          # Grid de parámetros + heatmap de robustez (IS)
│
├── risk/
│   ├── risk_manager.py       # Reglas de riesgo por instancia
│   ├── position_manager.py   # Gestión de posiciones
│   └── money_management.py   # Sizing, % de capital por trade
│
├── portfolio/
│   ├── pool.yaml             # Registro de instancias activas del portfolio
│   ├── loader.py             # Lee pool.yaml y carga cada instancia activa
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

### Siguientes
- [ ] **Sistema de instancias** — Estrategia base + config por activo (QQQ, SPY, IWM)
- [ ] **Pool de instancias** — `pool.yaml` + loader para registrar/activar instancias
- [ ] **Caché de cálculo** — `@st.cache_data` para recalcular al cargar sin lag
- [ ] **Botón "Exportar configuración"** — genera el YAML de los sliders para commitear a GitHub
- [ ] **Gestión de Riesgo** — Stop loss, riesgo máximo por operación (por instancia)
- [ ] **Gestión de Posiciones** — Seguimiento de trades abiertos y cerrados (por instancia)
- [ ] **Money Management** — Tamaño de posición basado en % de capital
- [ ] **Métricas (set ampliado)** — Sharpe, Sortino, Calmar, Profit Factor, Expectancy, exposure, etc.
- [ ] **Costos y ejecución** — Comisión/slippage estilo IBKR + precios de entrada/salida, por instancia
- [ ] **Benchmark Buy & Hold** — Comparación de cada instancia vs. mantener el activo
- [ ] **Backtest IS/OOS** — Múltiples bloques de fecha por conjunto, activables/excluibles, por instancia
- [ ] **Optimización** — Grid + heatmap de robustez en tab separada (IS)
- [ ] **Montecarlo** — Simulaciones de robustez por instancia
- [ ] **Filtros** — Eventos (`events.csv`), día de la semana y horario, por instancia con comparación con/sin
- [ ] **Portfolio Layer** — Drawdown unificado, correlación, capital total del pool
- [ ] **Expansión de activos** — SPY, IWM y large caps

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
  ⬜ Expansión de activos (SPY, IWM, large caps)
  ⬜ Segunda estrategia
```

---

*Documento vivo — se actualiza conforme evoluciona el proyecto.*
