# Handoff: Mapa de Cooperación Internacional

**Doctorado en Ciencias Sociales · Universidad Diego Portales (UDP)**

---

## Overview

Visualización interactiva de la red de cooperación internacional del programa: un mapa-mundi con pines geolocalizados de las instituciones extranjeras donde el doctorado mantiene **convenios institucionales** o donde **estudiantes** han realizado **pasantías de investigación**. El usuario puede:

- Filtrar por tipo (Convenios / Pasantías) — toggles independientes.
- Hacer zoom regional (Europa, América del Norte, América del Sur) o paneo libre.
- Ver detalles de cada institución vía tooltip al hacer hover.
- Consultar un listado completo agrupado por país debajo del mapa, sincronizado con los filtros activos.

## About the Design Files

Los archivos de este bundle son **referencias de diseño creadas en HTML/CSS/JS** — prototipos que muestran la apariencia y comportamiento deseado, **no código de producción para copiar directamente**.

La tarea es **recrear estos diseños en el entorno del codebase destino** (React, Vue, Next.js, SvelteKit, etc.) usando sus patrones y librerías establecidas. Si no existe un entorno previo, elegir el framework más apropiado para el proyecto e implementar allí.

El prototipo HTML usa **D3.js v7** + **TopoJSON** + **world-atlas v2 (countries-110m)**. Estas dependencias siguen siendo válidas en producción; pueden empaquetarse vía npm (`d3`, `topojson-client`, `world-atlas`).

## Fidelity

**High-fidelity.** Colores, tipografía, espaciado, transiciones y estados están definidos con precisión. El developer debe recrear el UI **pixel-perfect** usando las librerías existentes del codebase (por ejemplo: shadcn/ui + Tailwind si es React, o estilos equivalentes).

---

## Screens / Views

Es una **single-view application**: una página con tres bloques verticales.

### 1. Header (página)

- **Eyebrow** (encabezado pequeño): `DOCTORADO EN CIENCIAS SOCIALES · UDP`
  - 11px / weight 600 / letter-spacing 0.14em / uppercase
  - Color: `--mandy-700` (#9C3E4B)
- **Título H1**: `Mapa de cooperación internacional`
  - `clamp(22px, 2.4vw, 30px)` / weight 600 / letter-spacing -0.015em
  - La frase "cooperación internacional" va dentro de `<em>` con color `--mandy-600` (#BC4F5D), `font-style: normal`.
- **Subtítulo**: `Convenios institucionales y pasantías de investigación realizadas por estudiantes en universidades y centros de estudio en el extranjero.`
  - 14px / weight 400 / line-height 1.5 / `max-width: 720px`
  - Color: `--text-secondary` (rgba(28,28,28,0.62))

### 2. Map shell (`#wrap`)

Contenedor principal del mapa.

- `max-width: 1100px` / fondo blanco / `border: 1px solid rgba(28,28,28,0.08)` / `border-radius: 16px` / shadow medium
- Contenido:
  - **SVG del mapa** (D3 + TopoJSON)
    - `viewBox: 0 0 1100 540`
    - Proyección: `d3.geoNaturalEarth1().scale(178).translate([550, 288])`
    - Capas (en orden):
      1. Esfera (`<path datum={Sphere}>`) — fill `--bg-map-water` (#F7F5EE)
      2. Graticule — stroke `rgba(28,28,28,0.04)` / 0.4px
      3. Países — fill `--bg-map-land` (#E4E0D5) / stroke `--bg-map-water` / 0.6px
      4. Halos por país (círculos suaves bajo cada cluster) — fill `--mandy-500` / opacity 0.10
      5. Pines (ver §Pin geometry)
  - **Controles de zoom** (top-right, `z-index: 10`, columna vertical, gap 4px):
    - `EU` — botón región Europa: `zoomTo([10, 50], 4.2)`
    - `AN` — botón región América del Norte: `zoomTo([-95, 38], 3.0)`
    - `AS` — botón región América del Sur: `zoomTo([-62, -18], 2.8)`
    - separador 6px
    - `+` `−` `↺` — zoom in / out / reset
    - Cada botón: 34×34px / fondo blanco / border 1px soft / radius 6px / shadow-sm
    - Hover: fondo `--mandy-50`, color `--mandy-700`, border `--mandy-200`
  - **Hint chip** (bottom-center): `Scroll para hacer zoom · Arrastra para mover`
    - Auto-fade después de 4.2s (`opacity: 0` con transition 1.2s)
- **Bottom bar** (filtros + meta):
  - Padding 14px 18px / border-top 1px / fondo blanco
  - Izquierda: label `MOSTRAR` + botón `Convenios` + botón `Pasantías`
  - Derecha: `<n> instituciones visibles`

### 3. Pin geometry

Pines vectoriales en SVG.

- **Path**: `M0,0 C-3.6,-5.2 -7.8,-8.8 -7.8,-14 C-7.8,-19.2 -4.4,-21.8 0,-21.8 C4.4,-21.8 7.8,-19.2 7.8,-14 C7.8,-8.8 3.6,-5.2 0,0Z`
- **Punto interno** (circle): `cy=-14.5, r=3`, fill blanco
- **Tipos**:
  - `pin-c` (Convenio): fill `--mandy-500` / stroke blanco 1.4px
  - `pin-p` (Pasantía): fill `--sunset-600` (#ED2615) / stroke blanco 1.4px
  - `pin-both` (Convenio + Pasantía): fill `--mandy-500` / stroke `--sunset-600` 2px
- **Hover**: `transform: translateY(-2px) scale(1.08)` + drop-shadow
- **Filtro oculto** (`.dim`): `opacity: 0; transform: scale(0.6); pointer-events: none;` con transition 240ms ease

**Clustering**: cuando varias instituciones tienen el mismo país (≥2 pines en cercanía), se distribuyen en un anillo circular alrededor del centroide para evitar overlap. Ver función `clusterOffsets()` en el HTML.

### 4. Tooltip (`#tt`)

Popup absolute-positioned con detalles al hacer hover sobre un pin.

- Fondo blanco / border 1px medium / radius 10px / shadow-lg
- Padding 12px 14px / max-width 260px / min-width 180px
- **Estructura**:
  - `tn`: nombre institución — 13.5px / weight 600 / letter-spacing -0.005em
  - `tc`: país — 11px / weight 500 / uppercase / letter-spacing 0.08em / color terciario
  - `tgs`: tags (Convenio / Pasantía) — pills 10.5px weight 600
    - tag-c: bg `--mandy-100` (#F4D9DD) / color `--mandy-800` (#6F2C36)
    - tag-p: bg `#FFE5E1` / color `--sunset-700` (#C81C0D)
  - `tl`: link "Visitar sitio →" — color `--mandy-600`, hover `--mandy-800`
- Posicionamiento dinámico que evita salirse del wrap (offset 16px, flip cuando colisiona).

### 5. Filter buttons (`fbtn`)

Pill buttons toggle.

- Padding 7px 13px 7px 10px / border 1.5px medium / radius 999px
- 12.5px / weight 500
- Contenido: `<span class="fdot"></span> Convenios <span class="fcount">N</span>`
- **Estado off** (`aria-pressed="false"`): fondo blanco, color secundario, fdot tipo-color
- **Estado on** (`aria-pressed="true"`):
  - Convenios: fondo `--mandy-600`, border `--mandy-600`, texto blanco
  - Pasantías: fondo `--sunset-600`, border `--sunset-600`, texto blanco
- Hover (cuando off): border y texto pasan a `--text-primary`
- **Importante**: ambos toggles son **independientes**. Se permite tener los dos apagados (en cuyo caso el listado muestra estado vacío). Esto es comportamiento solicitado por el cliente.

### 6. Stats strip (4 cards)

Grid 4 columnas (2 en mobile <720px) — gap 10px.

- Cada card: bg blanco / border soft / radius 10px / padding 14px 16px
- Label: 10.5px weight 600 uppercase letter-spacing 0.1em color terciario
- Value: 22px weight 600 letter-spacing -0.02em
- Cards: `Países` · `Convenios` (accent mandy) · `Pasantías` · `Total instituciones`

### 7. List section (listado desplegable por país)

Sección debajo del mapa que muestra todas las instituciones agrupadas por país, sincronizada con los filtros activos.

- Container: bg blanco / border soft / radius 16px / padding 22px 24px 24px / margin-top 28px
- **Title**: `Listado de instituciones` — 17px weight 600
- **Subtitle**: `Filtrá usando los botones del mapa. El listado refleja la selección activa.` — 12.5px weight 400
- **Grid**: `grid-template-columns: repeat(auto-fill, minmax(260px, 1fr))` / gap 22px 28px
- Cada **country-block**:
  - H3 (`country-name`): nombre país — 11px weight 700 uppercase letter-spacing 0.12em color `--mandy-700`, border-bottom 1px `--mandy-100`
  - Badge `cn-count`: cantidad de instituciones en ese país — pill pearlbush
  - `<ul>` instituciones (sin bullet):
    - Cada `<li>`: padding 6px 0, border-bottom 1px dashed soft (excepto última)
    - Link al sitio (target="_blank") — 13px weight 500 hover color `--mandy-700` underline
    - Tags Convenio/Pasantía (mismas pills que el tooltip)
- **Empty state**: cuando ambos filtros están off → `Sin instituciones para mostrar. Activá Convenios o Pasantías.` (centrado, color terciario, 32px padding)

### 8. Footer

Línea fina con créditos.

- 11px weight 500 color terciario
- Izquierda: `Facultad de Ciencias Sociales e Historia · Universidad Diego Portales`
- Derecha: `Actualizado 2026`

---

## Interactions & Behavior

### Filtros
- Los botones `Convenios` y `Pasantías` son **toggles independientes** (no radio). Estado en memoria: `{ c: bool, p: bool }`, ambos `true` por defecto.
- Al togglear:
  1. Actualizar `aria-pressed` del botón clickeado.
  2. Llamar `applyFilter()` → recorre todos los pines, agrega/quita la clase `.dim` según `(pinIsConvenio && state.c) || (pinIsPasantía && state.p)`.
  3. Recalcular halos: si ningún pin del país queda visible, el halo se atenúa.
  4. Actualizar contador `<n> instituciones visibles`.
  5. Llamar `renderList()` → re-renderiza el listado de país agrupado, omitiendo países cuyas instituciones no pasan el filtro, y actualiza tags por institución.
- **Una institución que es Convenio + Pasantía (`k === "cp"`) sigue visible si CUALQUIERA de los dos filtros está activo.**

### Zoom y pan
- D3 zoom con `scaleExtent([1, 14])`.
- Botones EU/AN/AS hacen `transition().duration(700)` hacia las coords [lon, lat] y escala definida.
- Botones `+` / `−` usan `zoom.scaleBy(1.5)` / `zoom.scaleBy(0.67)` con duration 280ms.
- `↺` resetea a `d3.zoomIdentity` con duration 700ms.
- **NO counter-escalar los pines durante el evento de zoom.** En la versión inicial se intentó (`scale(1/sqrt(k))` aplicado a cada pin en cada evento) y produjo jitter/vibración perceptible en el frame. Los pines crecen junto al mapa — comportamiento estable y fluido. Si en producción se desea limitar el tamaño máximo del pin, hacerlo con `vector-effect: non-scaling-stroke` solo en el stroke, o reducir el path SVG en `transform-box: fill-box` controlado por una CSS var atada al nivel de zoom — pero NO recomputar en cada tick de zoom event.

### Tooltip
- Aparece en `mousemove`, desaparece en `mouseleave`.
- Posicionamiento relativo al `wrap`: offset (+16, -18) por defecto; si tooltip excede borde derecho (>width-280) se voltea a la izquierda; igual para borde inferior.
- Click en pin abre `d.u` en nueva pestaña (`window.open(url, '_blank', 'noopener')`).

### Animaciones / Transitions
- `--ease: cubic-bezier(0.4, 0, 0.2, 1)`
- Pin hover: 200ms transform + filter
- Pin filter on/off: 240ms opacity + transform
- Hint chip fade-out: 1.2s opacity
- Filter button hover: 200ms all
- Zoom buttons hover: 160ms background/color/border

---

## State Management

```ts
type Kind = 'c' | 'p' | 'cp';
interface Institution {
  n: string;        // nombre
  c: string;        // país
  ll: [number, number]; // [lon, lat]
  u: string;        // URL
  k: Kind;          // tipo: convenio / pasantía / ambos
}

interface FilterState {
  c: boolean;       // mostrar convenios
  p: boolean;       // mostrar pasantías
}
// Inicial: { c: true, p: true }
```

No requiere data fetching dinámico — el dataset es estático y vive en el código.

**Datos externos**:
- TopoJSON del mundo: `world-atlas@2/countries-110m.json` (~110KB). Recomendable empaquetar vía npm o servir desde CDN propia. Cargar 1 vez al montar el componente.

---

## Design Tokens

### Colors

**Mandy (primario · facultad)**
| Token | Hex |
|---|---|
| --mandy-50  | #FBF0F2 |
| --mandy-100 | #F4D9DD |
| --mandy-200 | #EBB8C0 |
| --mandy-300 | #E29CA6 |
| --mandy-400 | #DA808E |
| --mandy-500 | #D36473 ← base |
| --mandy-600 | #BC4F5D |
| --mandy-700 | #9C3E4B |
| --mandy-800 | #6F2C36 |
| --mandy-900 | #461C22 |
| --mandy-950 | #2A1014 |

**Neutros**
| Token | Hex |
|---|---|
| --woodsmoke-950 | #1C1C1C |
| --pearlbush-200 | #E4E0D5 |
| --pearlbush-100 | #EFEDE5 |
| --pearlbush-50  | #F7F5EE |
| --white         | #F4F4F4 |
| --pure-white    | #FFFFFF |

**Sunset (acento · pasantías / alerta)**
| Token | Hex |
|---|---|
| --sunset-500 | #FF5B4D |
| --sunset-600 | #ED2615 ← pin pasantía |
| --sunset-700 | #C81C0D |

**Semánticos**
| Token | Valor |
|---|---|
| --bg-page | #F4F4F4 |
| --bg-surface | #FFFFFF |
| --bg-map-land | #E4E0D5 |
| --bg-map-water | #F7F5EE |
| --border-soft | rgba(28,28,28,0.08) |
| --border-medium | rgba(28,28,28,0.14) |
| --text-primary | #1C1C1C |
| --text-secondary | rgba(28,28,28,0.62) |
| --text-tertiary | rgba(28,28,28,0.44) |
| --color-convenio | --mandy-500 |
| --color-pasantia | --sunset-600 |

### Typography

- **Familia**: `Work Sans` (Google Fonts), pesos 300, 400, 500, 600, 700
- **Default body**: weight 500 (Medium)
- Fallback stack: `'Work Sans', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`
- Smoothing: `-webkit-font-smoothing: antialiased`
- `font-feature-settings: "ss01", "cv11"`

### Spacing / radius / shadow

- Radius: 6 / 10 / 16 / 999px
- Shadows:
  - `--shadow-sm`: `0 1px 2px rgba(28,28,28,0.04), 0 2px 6px rgba(28,28,28,0.04)`
  - `--shadow-md`: `0 2px 6px rgba(28,28,28,0.06), 0 8px 24px rgba(28,28,28,0.08)`
  - `--shadow-lg`: `0 8px 28px rgba(28,28,28,0.12)`
- Easing global: `cubic-bezier(0.4, 0, 0.2, 1)`

---

## Database (instituciones)

30 instituciones en 13 países. Schema arriba (§State Management).

| País | Instituciones |
|---|---|
| Alemania | GIGA · Max Planck Institute · U. de Hildesheim · U. Libre de Berlín |
| Argentina | U. de San Andrés · UNSAM · UTDT |
| Brasil | USP · U. de Brasilia |
| Canadá | U. de Montreal |
| Colombia | U. del Rosario |
| EE. UU. | American University |
| España | UAM · UAB · Complutense · U. de Salamanca · Pompeu Fabra |
| Francia | Sciences Po Lille · Sciences Po Paris · Panthéon-Sorbonne · Paris 9 (Dauphine) · Paris 10 Nanterre · Sorbonne Nouvelle |
| México | CIDE · UAM México |
| Países Bajos | U. de Ámsterdam · U. de Leiden · VU Amsterdam |
| Perú | PUCP |
| Uruguay | U. Católica del Uruguay |

Tipo (`k`) e instituciones marcadas como `cp` (Convenio + Pasantía) están en el dataset embebido en el HTML — copiarlo tal cual.

---

## Assets

- **Tipografía**: Work Sans (Google Fonts) — `https://fonts.googleapis.com/css2?family=Work+Sans:wght@300;400;500;600;700&display=swap`
- **Mapa base**: world-atlas v2 countries-110m.json — `https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json` (o vía npm: `npm i world-atlas`)
- **Librerías**: `d3@7`, `topojson-client@3`
- **Sin imágenes** (todo SVG inline / generado por D3)

---

## Files

- `Mapa Cooperacion UDP.html` — fuente del prototipo (HTML + CSS + JS, ~830 líneas). Esta es la referencia principal.
- `Mapa Cooperacion UDP · standalone.html` — bundle self-contained (incluye librerías y atlas inlineados, ~352KB). Útil para abrir offline y comparar pixel-perfect.

---

## Implementation tips

1. **Componentización sugerida** (React):
   - `<MapaCooperacion />` (raíz)
     - `<Header />`
     - `<MapShell>` que envuelve `<WorldMap>` + `<ZoomControls>` + `<HintChip>` + `<FilterBar>` + `<Tooltip>`
     - `<StatsStrip />`
     - `<InstitutionList />`
     - `<Footer />`
2. Usar `react-d3-graph` o D3 directo dentro de un `useEffect`/`useRef` para SVG. Evitar declarar el SVG en JSX puro (no escala bien con clustering programático).
3. **Estado global**: dos booleans (`showConvenios`, `showPasantias`). Mover a Context si crece la app.
4. **Accesibilidad**:
   - Botones de filtro tienen `aria-pressed`.
   - Tooltip tiene `role="tooltip"` y `aria-live="polite"` (queda como mejora — considerar también keyboard focus en pines).
   - Pines no son focuseables vía teclado en el prototipo. **Mejora pendiente**: agregar `tabindex="0"` y handlers de Enter/Space.
5. **Responsive**: el mapa mantiene aspect ratio vía `viewBox`. En mobile <720px: stats grid pasa de 4 a 2 columnas, label "Mostrar" se oculta.

---

## Open questions / decisiones pendientes

- ¿Pop-ups al click (no solo hover) con info extendida? El prompt original lo sugiere — actualmente click abre el sitio externo. Confirmar con cliente.
- ¿Persistir estado de filtros en URL/localStorage? No implementado.
- ¿Soporte de teclado completo en el mapa (focus + Enter)? Mejora de accesibilidad recomendada.
