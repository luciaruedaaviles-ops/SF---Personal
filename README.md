# Torre de Control Comercial · Pipe con Productos

Tablero web (`index.html`, autocontenido, sin backend) que replica el diseño y la lógica de
**"Torre de Control Comercial · SONDA México"**, adaptado a la fuente de datos real disponible:
el reporte de Salesforce **"Pipe con productos FCST"** (p. ej. `FCST-2026-09-02-10-30-41-carga.xlsx`).

## Cómo usarlo

1. Abre `index.html` en cualquier navegador (doble clic, o súbelo a GitHub Pages / cualquier
   hosting estático — no necesita servidor ni build).
2. Arrastra el Excel exportado esa semana. Debe traer dos hojas en el mismo libro:
   - **`Cuotas`** — bloques "CUOTAS \<año\> POR VERTICAL" y "CUOTAS \<año\> POR HORIZONTAL", uno
     por año.
   - **`Pipe con productos FCST`** — el reporte de oportunidades con una fila por línea de
     producto (columnas como `ID Oportunidad`, `Etapa`, `Estatus`, `Monto (convertido)`, etc.).
3. Todo el cálculo ocurre en el navegador (SheetJS + Chart.js, cargados desde CDN). Nada se
   sube a ningún servidor.
4. Para cargar el corte de otra semana, usa el botón **"Cargar nueva semana"** — no hace falta
   recargar la página.

## Filtro de entrada

Todos los análisis excluyen cualquier fila cuyo **`Tipo de registro de la oportunidad`** no sea
exactamente `Oportunidad Estándar` u `Oportunidad Fast Track` — quedan fuera las variantes
`Oportunidad Estándar Secundaria` y `Oportunidad Fast Track Secundaria` (venta indirecta). Se
aplica una sola vez al leer el archivo, así que alcanza de forma uniforme a todas las secciones.
El número de líneas excluidas se muestra en el pie del tablero.

## Qué calcula

- **Corte Mensual**: Cuota, Ganada, Comprometida, Forecast (Ganada+Comprometida), Pipe Abierto
  (Comprometida+Probable+Indeterminada) y Gap vs. Cuota, por mes/año/vertical, con gráfico de
  barras Cuota vs. Ganada vs. Forecast por Vertical.
- **Cobertura vs. piso mínimo**: Pipe Abierto ÷ Cuota H2 (Jul–Dic), con gauge y desglose por
  Vertical (piso 3x, meta 4x).
- **Ganada vs. Presupuesto**: dos tablas independientes — por Vertical y por Línea de Servicio
  (ver "Decisiones de diseño" abajo, por qué no están cruzadas).
- **Pipe abierto por KAM**: ranking por Pipe Abierto, con Ganado del año y # de oportunidades
  abiertas.
- **Composición del Pipe**: heatmap Estatus (Comprometida/Probable/Indeterminada) × Vertical ×
  Línea de Servicio.
- **Pipe abierto por Producto**: ranking de familias y productos por Pipe Abierto — el
  diferenciador de esta fuente de datos frente al workbook original.
- **Radar de Riesgo**: oportunidades abiertas **vencidas** (Fecha Real de Cierre ya pasada) o
  **estancadas** (días en la etapa actual muy por encima de la mediana del pipe abierto).
- **Cerradas por Marca**: Ganada vs. Perdida por Marca (columna `Marca`, por línea de producto),
  solo oportunidades ya decididas del año. "SONDA" (marca propia) se destaca aparte de las
  marcas de terceros — sigue en la tabla completa, pero se excluye del gráfico porque su escala
  (10-100x mayor que cualquier marca revendida) aplastaría el resto a líneas invisibles.

## Decisiones de diseño (por qué difiere del tablero original de SONDA)

El tablero original leía un workbook curado de 6 hojas (`Consolidado`, `Cuotas (V)`,
`Cuotas (H)`, `PAIS H1+Q3`, `PROD. KAM`, `Secundarias`, `Detalle1`) con datos que este reporte
plano de Salesforce no trae. Se tomaron estas decisiones, confirmadas con el usuario:

- **Sin Best Case**: el archivo no trae el campo booleano `Upside` que definía Best Case en el
  original. Se muestran solo Ganada, Comprometida, Forecast y Pipe Abierto.
- **Sin Cuota por KAM**: la hoja `Cuotas` solo trae metas por Vertical y por Línea de Servicio,
  no por KAM individual (el original la leía de una hoja `PROD. KAM` que no existe aquí). La
  tabla de KAM muestra Ganado y Pipe Abierto, sin columna de Cuota/GAP.
- **Sin Movimiento Semanal**: este archivo es un corte único (no trae histórico semana a
  semana). Las secciones de tendencia del original no están replicadas.
- **Ganada vs. Presupuesto en dos tablas separadas**: la hoja `Cuotas` trae metas de Vertical y
  de Línea de Servicio como totales nacionales independientes, no cruzados (a diferencia del
  original, que sí tenía el detalle línea por línea para cruzarlos). Cruzarlos aquí habría
  significado inventar una distribución que el archivo no sustenta.
- **Línea de Servicio agrupada**: el campo crudo `Delivery Vertical/Línea de Servicios` de
  Salesforce es muy granular (25+ variantes). Se agrupó en un set curado de ~9 categorías
  legibles (ver `HORIZ_MAP` en el código) — mismo criterio que el `Grupo LS` ya curado del
  tablero original, pero construido aquí porque este archivo no lo trae.
- **Radar de Riesgo**: sección nueva, no existía en el original. Usa los campos
  `Duración de la etapa` y `Fecha Real de Cierre` (que si vienen en este reporte) para señalar
  oportunidades estancadas o vencidas — el umbral de "estancada" se recalcula automáticamente
  como 2× la mediana de días en etapa del pipe abierto filtrado.

## Estructura

Un único archivo HTML (`index.html`) con CSS y JS inline. Librerías externas cargadas por CDN:
[SheetJS](https://sheetjs.com/) (lectura de Excel) y [Chart.js](https://www.chartjs.org/)
(gráficos). Sin dependencias de build ni Node — es una página estática.

## Reporte: Oportunidades ≥ USD 500K · Seguimiento de Deal (`oportunidades-500k.html`)

Página aparte, del mismo tipo (estática, sin servidor), que lee el mismo Excel semanal de
"Pipe con productos FCST" y responde: de las oportunidades de USD 500K o más, ¿cuántas ya
tuvieron deal y cuántas no?

- **Filtros** (una fila arriba): Unidad Comercial, año de `Fecha de creación`, Etapas (solo
  abiertas, por omisión, o incluir cerradas) y búsqueda por ID, cuenta o tema.
- **Universo de todo el tablero**: oportunidades **con deal de cualquier monto** + oportunidades
  **sin deal con Monto (convertido) ≥ 500K**. Lo usan por igual el banner, los KPIs, el cuadro
  resumen y las tablas de detalle, así que todo cuadra entre sí.
- **Banner ejecutivo y KPIs**: total ≥ 500K, con deal, sin deal (# y monto) y % con deal.
- **Cuadro resumen**: Unidad Comercial → Etapa en filas; `SI`, `SI - Corporativo`, `Sin deal`,
  Total y % con deal en columnas, con barra de distribución y monto. Clic en una unidad abre sus
  etapas; al elegir una unidad en el filtro se abre sola.
- **¿Cómo se llega al resumen?**: conciliación paso a paso para la unidad elegida: todas las
  oportunidades únicas → universo del tablero (con deal cualquier monto + sin deal ≥ 500K) →
  abiertas, con cuántas tienen deal en cada paso.
- **Con deal**: todas las oportunidades con "SI" en `Se agendo Deal`, **de cualquier monto**.
  Trae un resumen por `Vertical` (col. AC) × Etapa con cantidad y monto, y el detalle (fecha de
  creación, días, ID, cuenta, tema, etapa, vertical, tipo de deal y monto).
- **Sin deal**: solo las de **Monto (convertido) ≥ 500K**, con el mismo detalle.
- Las dos tablas respetan Unidad Comercial, año, Etapas y búsqueda, y son ordenables por
  cualquier columna.
- **Descargar Excel**: hojas Resumen, Con deal y Sin deal con el filtro activo.

Reglas:
- Valores absolutos: se cuenta **una vez cada `ID Oportunidad`** (col. B). Monto, etapa y deal se
  repiten igual en cada línea de producto, así que no se suman líneas (una tabla dinámica de
  Excel con "Cuenta de Precio total" cuenta líneas, por eso da números mayores).
- Sin deal: `Monto (convertido)` (col. BF) ≥ 500,000, comparado en valor absoluto. Con deal:
  cualquier monto.
- `Tipo de registro de la oportunidad` (col. Y) = `Oportunidad Estándar` u `Oportunidad Fast Track`.
- `Se agendo Deal` (col. BH): cualquier valor que empiece con "SI" = con deal; vacío = sin deal.
- Abiertas = `Etapa` (col. D) distinta de `Cerrada Ganada` y `Cerrada Perdida`.
- Las columnas se ubican por el nombre del encabezado; si el nombre cambia, se usa la letra.
  Al pie del reporte se muestran las columnas que se leyeron.
- Formato de las tablas: cantidad y monto en una sola celda, con el monto en millones de USD
  (`2 / $3.0M`). El detalle por oportunidad muestra el monto en millones con 2 decimales. La
  descarga a Excel conserva cantidad y monto en columnas separadas, con el monto completo.
