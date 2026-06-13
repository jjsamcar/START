# Estrategia 1 — Overnight Gap Trading

> Documento de análisis y especificación de la estrategia. Parte del [JJ Trading System](../../JJ_Trading_System.md).

**Estado**: 🟡 En definición (conforme avanza la lectura)

---

## Resumen

| Campo | Detalle |
|---|---|
| **Nombre** | Overnight Gap Trading |
| **Tipo** | Gap entre cierre y apertura (overnight) |
| **Activos** | QQQ, SPY, IWM (índices ETF) |
| **Timeframe** | Diario |
| **Lógica base** | `strategies/overnight_gap.py` |
| **Configs** | `strategies/configs/overnight_gap_<TICKER>.yaml` |

---

## Concepto

Por definir conforme avance la lectura del documento de la estrategia.

*(Espacio para anotar la idea central: qué es el gap overnight, por qué genera oportunidad, sesgo direccional, etc.)*

---

## Instancias (una config por activo)

Cada activo se opera como una instancia independiente con sus propios parámetros. Misma lógica base, distinta configuración.

| Instancia | Activo | Estado | Notas |
|---|---|---|---|
| `OG_QQQ` | QQQ | En definición | Datos CSV disponibles |
| `OG_SPY` | SPY | Pendiente | Falta descargar datos |
| `OG_IWM` | IWM | Pendiente | Futuro |

---

## Parámetros

### Filtros de señal

| Parámetro | Descripción | Control en dashboard |
|---|---|---|
| `gap_min` | Tamaño mínimo del gap para considerar la señal | Slider de rango |
| `gap_max` | Tamaño máximo del gap para considerar la señal | Slider de rango |

> **Filtro por rango de gap**: el usuario puede filtrar las operaciones según el tamaño del gap (ej. solo gaps entre 0.5% y 2%). El valor vive en la config de la estrategia; el slider que lo mueve vive en el dashboard. La lógica recibe `gap_min`/`gap_max` y filtra las señales.

*(Más parámetros por definir: dirección del gap, día de la semana, etc.)*

---

## Reglas de Entrada

Por definir.

---

## Reglas de Salida

Por definir.

---

## Gestión de Riesgo (por instancia)

Por definir. Cada instancia (estrategia+activo) tendrá su propio:
- Stop loss
- Tamaño de posición / sizing
- Riesgo máximo por operación

---

## Notas de Lectura

*(Aquí se van anotando ideas, observaciones y citas del material que estás estudiando.)*

---

*Documento vivo — se actualiza conforme evoluciona el análisis.*
