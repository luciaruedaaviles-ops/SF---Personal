# Torre de Control Comercial · Pipe con Productos

Este repositorio tiene dos tableros web independientes, ambos autocontenidos (sin backend), que
leen el mismo reporte de Salesforce **"Pipe con productos FCST"**:

- **`index.html`** — Torre de Control Comercial: vista ejecutiva con KPIs, cobertura, cumplimiento
  de presupuesto, ranking por KAM/Producto y radar de riesgo. Ver detalle más abajo.
- **`tablero-fcst.html`** — Revisión Semanal: réplica exacta de las tablas dinámicas de la hoja
  "TD" (+ "Secundarias" y "MegaDeals") del workbook de seguimiento semanal, pensada para revisar
  el corte cada 8 días usando la **columna A ("Fecha de extracción")** de la hoja "Pipe con
  productos FCST" como filtro de corte. Ver `tablero-fcst.html` — sección "Cómo usarlo" más abajo.

## Torre de Control Comercial (`index.html`)

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

---

## Revisión Semanal (`tablero-fcst.html`)

Segundo tablero, independiente del anterior: en vez de un resumen ejecutivo por KPIs, replica
**tal cual** las tablas dinámicas que ya se revisan cada semana en el workbook de seguimiento
(hoja "TD" con 8 tablas dinámicas, más "Secundarias" y "MegaDeals"), recalculadas en vivo desde
las filas crudas de la hoja "Pipe con productos FCST" — no lee los valores ya cacheados de las
tablas dinámicas del Excel (esos valores quedan "congelados" desde el último `Actualizar todo`
en Excel; aquí siempre se recalculan con los datos más recientes del archivo).

### Cómo usarlo

1. Abre `tablero-fcst.html` en el navegador y sube el Excel de "Pipe con productos FCST".
2. Si el archivo acumula varias semanas en una sola hoja, llena la **columna A** de la hoja
   "Pipe con productos FCST" con la **fecha de extracción** de cada corte (la misma fecha para
   todas las filas de esa semana). El tablero detecta las fechas distintas en esa columna y arma
   una pestaña por corte — así puedes revisar el corte de hoy o volver a uno anterior cada 8 días.
3. Si la columna A todavía está vacía (como en un export normal de Salesforce), el tablero usa
   automáticamente la fecha de generación del reporte ("A partir de..." en el encabezado) como
   único corte — no hace falta preparar nada para la primera semana.
4. El selector "Año R" aplica a todas las tablas de Forecast/Pipe/Perdida (por Fecha Real de
   Cierre), igual que en el workbook original donde cada año tiene su propio bloque de tablas.

### Tablas replicadas y sus filtros

Cada tarjeta muestra debajo del título los filtros exactos que aplica (igual que la tabla
dinámica de origen). En resumen:

| Tarjeta | Filas | Columnas | Filtros clave |
|---|---|---|---|
| Cerrada Ganada (sin Mega Deals) | Estatus | Mes R | Tipo Estándar/Fast Track · Mega Deal = No · Etapa = Cerrada Ganada |
| Cerrada Ganada · Digital Operations | Estatus | Mes R | Igual + Delivery Vertical solo líneas "Digital Operations" |
| Cerrada Perdida | Estatus | Mes R | Mega Deal = Todas · Etapa = Cerrada Perdida |
| Pipe Abierto (sin Mega Deals) | Estatus | Mes R | Mega Deal = No · Etapa = Prospección/Solución/Negociación/Cierre |
| Pipe Abierto (con Mega Deals) | Estatus | Mes R | Mega Deal = Todas · mismas etapas abiertas |
| Pipe Abierto por Vertical | Estatus × Vertical | Mes R | Mega Deal = Todas · Delivery Vertical = Todas |
| Creación de Oportunidades # / $ | Estatus | Día de creación | Mes/Año de creación = mes de la Fecha de extracción del corte activo |
| Venta Secundaria — Cerrada Ganada | Unidad de Comercial | Delivery Vertical/Línea de Servicios | Tipo = Secundaria (venta indirecta) · Etapa = Cerrada Ganada |
| Mega Deals — Detalle | — (lista por oportunidad) | — | Mega Deal = Sí, agrupado por ID de oportunidad |

En todas las tablas el valor es la **suma de "Precio total (convertido)"** por línea de producto
(sin deduplicar por oportunidad, igual que la tabla dinámica origen), salvo en "Mega Deals" donde
sí se agrupa por oportunidad (una oportunidad puede tener varias líneas de producto) y en
"Creación de Oportunidades #" donde el valor es un conteo de líneas.

Cada tarjeta además muestra, cuando hay al menos dos fechas de extracción cargadas, la variación
del total general contra el corte anterior — la comparación que se necesita para una revisión
cada 8 días.

### Estructura

Igual que `index.html`: un único archivo HTML con CSS y JS inline, [SheetJS](https://sheetjs.com/)
por CDN, cálculo 100% en el navegador, sin envío de datos a ningún servidor.
