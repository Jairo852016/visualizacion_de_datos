# Señales de Alpha en Portafolios de ETFs

Proyecto final de la materia **Visualización de Datos** (Maestría). Sitio web interactivo que analiza el poder predictivo de señales técnicas —momentum, reversión a la media y volatilidad— sobre un universo de 15 ETFs entre 2015 y 2024.

---

## ¿Qué hace este proyecto?

El sitio responde cuatro preguntas analíticas usando el **Information Coefficient (IC)** como métrica central:

| # | Pregunta |
|---|---|
| P1 | ¿Qué señal tiene mayor poder predictivo por clase de activo? |
| P2 | ¿Cómo varía la efectividad de las señales a lo largo del tiempo? |
| P3 | ¿Existe correlación entre señales que permita diversificación? |
| P4 | ¿Una combinación simple de señales supera a señales individuales? |

Cada pregunta tiene su propia visualización interactiva construida con **Altair** (Vega-Lite).

---

## Stack Tecnológico

| Herramienta | Versión recomendada | Uso |
|---|---|---|
| [Quarto](https://quarto.org) | ≥ 1.4 | Framework del sitio web |
| Python | ≥ 3.10 | Kernel de los notebooks `.qmd` |
| yfinance | ≥ 0.2 | Descarga de precios históricos |
| pandas | ≥ 2.0 | Manipulación de datos |
| numpy | ≥ 1.24 | Cálculos numéricos |
| scipy | ≥ 1.11 | Correlación de Spearman (IC) |
| altair | ≥ 5.0 | Todas las visualizaciones |
| pyarrow | ≥ 14.0 | Lectura/escritura de Parquet |

> Solo se usa **Altair** para visualizaciones. No se usa plotly, matplotlib ni seaborn.

---

## Estructura del Proyecto

```
visualizacion_de_datos/
│
├── _quarto.yml           # Configuración del sitio (navbar, tema, output)
├── theme.css             # Tema visual personalizado (oscuro, paleta ámbar/teal)
├── favicon.svg           # Ícono del sitio (letra α en ámbar)
├── CLAUDE.md             # Contexto completo del proyecto para Claude Code
│
├── index.qmd             # Página principal: intro, universo de ETFs, metodología
├── data_cleaning.qmd     # Paso 1: descarga yfinance → limpieza → cálculo de señales
├── visualizations.qmd    # Paso 2: 4 preguntas → 4 charts Altair interactivos
│
├── data/
│   ├── raw/              # Precios descargados (generados al renderizar)
│   └── processed/
│       └── signals.parquet  # Dataset final con señales calculadas (~450k filas)
│
└── docs/                 # Sitio renderizado — publicado en GitHub Pages
    ├── index.html
    ├── data_cleaning.html
    ├── visualizations.html
    └── site_libs/
```

---

## Dataset

### Universo de ETFs (15 activos)

| Clase de Activo | Tickers |
|---|---|
| Renta Variable US | SPY, QQQ, IWM |
| Internacional | EFA, EEM, VWO |
| Renta Fija | TLT, IEF, LQD, HYG |
| Commodities | GLD, USO, DBC |
| Real Estate | VNQ |
| Volatilidad | VXX |

- **Período:** 2015-01-01 a 2024-12-31
- **Frecuencia:** Diaria
- **Variable base:** Precio de cierre ajustado (`Adj Close`, corregido por dividendos y splits)

### Señales Técnicas

| Señal | Descripción | Ventana |
|---|---|---|
| `mom_1m` | Retorno pasado 21 días hábiles | 21d |
| `mom_12m` | Retorno pasado 252 días, excluyendo el último mes | 252d |
| `mean_rev` | Z-score del precio respecto a su media móvil | 20d |
| `vol_21` | Volatilidad realizada (desv. estándar de retornos) | 21d |
| `rel_vol` | Volumen relativo vs promedio histórico | 20d |
| `corr_spy` | Correlación rolling con el ETF SPY | 60d |

### Variable Objetivo

`fwd_ret_21d` — retorno acumulado de los **próximos** 21 días hábiles. Es la variable que las señales intentan predecir. Se calcula con `shift(-21)` para evitar look-ahead bias.

---

## Flujo de Datos

```
yfinance  →  data_cleaning.qmd  →  signals.parquet  →  visualizations.qmd
  (raw)         (Paso 1)             (dataset)              (Paso 2)
```

1. `data_cleaning.qmd` descarga los precios, calcula las 6 señales y guarda el resultado en `data/processed/signals.parquet`.
2. `visualizations.qmd` carga el parquet y genera las 4 visualizaciones. No recalcula nada.

---

## Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/<tu-usuario>/<tu-repo>.git
cd visualizacion_de_datos
```

### 2. Crear entorno virtual (recomendado)

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
```

### 3. Instalar dependencias

```bash
pip install yfinance pandas numpy scipy altair pyarrow
```

### 4. Instalar Quarto

Descarga el instalador desde [quarto.org/docs/get-started](https://quarto.org/docs/get-started/) e instálalo según tu sistema operativo. Verifica la instalación:

```bash
quarto --version
```

---

## Cómo Ejecutar

### Renderizar el sitio completo

```bash
quarto render
```

Esto ejecuta el código Python de los tres `.qmd` en orden y genera los HTML en `docs/`.

> La primera ejecución descarga ~10 años de datos de yfinance y puede tardar 1–2 minutos. Las siguientes son más rápidas porque el parquet ya existe.

### Vista previa en local

```bash
quarto preview
```

Abre el sitio en tu navegador en `http://localhost:xxxx` con recarga automática al guardar cambios.

### Renderizar solo una página

```bash
quarto render data_cleaning.qmd
quarto render visualizations.qmd
```

---

## Visualizaciones

| Gráfico | Tipo | Interactividad |
|---|---|---|
| **P1 — Heatmap de IC** | `mark_rect` señal × clase | Tooltip con valor exacto |
| **P2 — IC Rolling** | `mark_line` + banda | Clic en leyenda para filtrar señal |
| **P3 — Correlación entre señales** | `mark_rect` señal × señal | Tooltip con correlación |
| **P4 — Sharpe Ratio comparativo** | `mark_bar` horizontal | Tooltip, ámbar = combo |

Todos los charts usan el tema oscuro personalizado (`custom_theme` de Altair) coherente con el CSS del sitio.

---

## Publicación en GitHub Pages

El sitio se publica desde la carpeta `docs/` en la rama `main`.

### Configuración en GitHub

1. Ve a **Settings → Pages** en tu repositorio
2. En **Source**, selecciona: `Deploy from a branch`
3. Rama: `main` / Carpeta: `/docs`
4. Guarda. El sitio estará disponible en `https://<usuario>.github.io/<repo>/`

### Flujo de actualización

```bash
quarto render          # genera docs/
git add docs/
git commit -m "update site"
git push
```

GitHub Pages despliega automáticamente al detectar cambios en `docs/`.

---

## Notas Metodológicas

- **Sin look-ahead bias:** cada señal en la fecha `t` usa únicamente datos anteriores a `t`. El retorno forward se calcula con `shift(-21)` y nunca se usa como input de ninguna señal.
- **IC (Information Coefficient):** correlación de Spearman entre señal y retorno forward. Valores típicos: −0.10 a +0.10. Un IC sostenido > 0.05 se considera económicamente significativo.
- **Señal combinada (P4):** promedio simple de z-scores normalizados cross-seccionalmente. Sin optimización ni overfitting.
- **Datos gratuitos:** `yfinance` no requiere API key. Los precios son ajustados por dividendos y splits (`auto_adjust=True`).

---

## Autor

**Jairo Pérez** — Maestría en Ciencia de Datos  
Materia: Visualización de Datos · 2026
