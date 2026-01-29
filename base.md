# Base — forma de trabajo y stack (Omnicity)

## Objetivo
Definir el marco de trabajo del repo para diseñar brochures y documentos informativos de Omnicity de forma reproducible, segura para agentes y lista para impresión.

## Alcance (por ahora)
- Definir stack técnico.
- Definir convenciones de diseño/impresión.
- Definir estructura de repo y flujo de trabajo.
- Aplazar la producción de piezas específicas hasta tener el pipeline listo.

## Mapa de herramientas/librerías (por capas)
- Autoría: HTML/CSS en VS Code.
- Templating: Nunjucks/Handlebars/Mustache (separa content.json del layout).
- Paginación: CSS Paged Media y Paged.js.
- PDF: Playwright (Chromium print), WeasyPrint, PrinceXML/Antenna House.
- QA: html-validate, stylelint, ESLint HTML plugin, Pa11y, visual regression.
- Optimización: qpdf, pdfcpu (post-procesado/validación).

## Stack recomendado (actual)
### Decisión principal: motor HTML/CSS -> PDF
#### A) Chromium print via Playwright (default para 1–4 páginas)
- Renderiza HTML/CSS igual que Chrome y exporta a PDF.
- Pros: fiel a navegador, fácil CI, rápido.
- Contras: headers/footers con particularidades (styles/fuentes no siempre heredan como esperas).
- Ideal para: brochures corporativos, fichas, afiches, 1–4 páginas.

**Dependencias clave**
- `playwright` (export PDF)
- `prettier` (formato)
- `stylelint` + config (higiene CSS)
- `eslint` (si hay JS para plantillas)
- Opcional: `vite` (preview rápido / bundling de assets)
- Opcional: `nunjucks`/`handlebars`/`mustache` (templating)

#### B) Paginación editorial con Paged.js
- Polyfill de CSS Paged Media y fragmented layout en el browser.
- Pros: control editorial (saltos, contadores, running content).
- Contras: edge cases; estilos `@media print` pueden afectar preview si no se encapsulan.
- Ideal para: documentos largos, catálogos multi-página, estilo revista.

#### C) Renderers pro de imprenta (pagos)
- PrinceXML y Antenna House Formatter.
- Pros: calidad editorial/prepress y features avanzados de Paged Media.
- Contras: licencias.
- Ideal para: entregables editoriales exigentes.

#### D) Python-first
- WeasyPrint (HTML/CSS -> PDF).
- Pros: pipeline Python, estable para documentos sistemáticos.
- Contras: no siempre igual a Chrome en CSS moderno.
- Ideal para: reportes, facturas, generación masiva.

**Decisión actual:** iniciar con **Playwright**. Si el documento crece en páginas o complejidad editorial, evaluar **Paged.js**.

## Paged Media (CSS) — base técnica mínima
- Dominar `@page`, `break-before`, `break-inside`, contadores, running headers/footers.
- Consultar guías de Paged Media (ej. docs de Prince y patrones de Paged.js) para casos avanzados.

## Estructura de repo (propuesta)
```
print-studio/
  packages/
    core/
      src/
        tokens/
          brand.tokens.json
        styles/
          base.css
          print.css
          components.css
        templates/
          brochure.html
          onepager.html
        scripts/
          render.mjs
          export-pdf.mjs
          validate.mjs
      package.json
  projects/
    omnicity-brochure-2026/
      content/
        es.json
        en.json
      assets/
        images/
        fonts/
        logos/
      src/
        index.html
        styles.css
      dist/
        omnicity-brochure-es.pdf
        omnicity-brochure-en.pdf
      project.config.json
  .github/workflows/
    pdf.yml
  README.md
```

## Convenciones de diseño para impresión (no negociables)
### Print CSS
- Definir hoja y márgenes con `@page`.
- Evitar cortes feos:
  - `break-inside: avoid;`
  - `break-before: page;`
- Usar unidades físicas en impresión: `mm`, `pt`, `cm`.

Ejemplo base:
```css
@page { size: A4; margin: 14mm; }
* { box-sizing: border-box; }
body { font: 11pt/1.45 system-ui; }
.break { break-before: page; }
.card, h2, ul { break-inside: avoid; }
```

### Tipografía
- Fuentes locales dentro del repo (`assets/fonts`) y declaradas con `@font-face`.
- Definir escala tipográfica (ej: 28pt, 16pt, 11pt, 9pt) y mantenerla.

### Imágenes
- Logos/íconos en SVG cuando sea posible.
- Fotos a 300 DPI equivalente según tamaño impreso.
- `assets/` organizado con nombres consistentes.

### Color (impresión real)
- Web es RGB; imprenta puede pedir CMYK/PDF/X.
- En práctica: exportar PDF desde Chromium; si la imprenta exige perfiles, ajustar pipeline con herramientas de prepress.

## Motor de contenido (tokens + contenido)
### Design tokens (fuente de verdad)
`brand.tokens.json`
```json
{
  "colors": { "ink": "#111111", "muted": "#666666", "border": "#E5E5E5" },
  "spacingMm": { "xs": 2, "sm": 4, "md": 6, "lg": 10 },
  "radiiMm": { "sm": 2, "md": 3 },
  "typePt": { "h1": 28, "h2": 14, "body": 11, "small": 9 }
}
```

Los scripts convierten tokens -> variables CSS:
```css
:root{
  --ink:#111;
  --space-md:6mm;
  --h1:28pt;
}
```

### Contenido en JSON
`content/es.json`
```json
{
  "title": "OMNICITY 2026",
  "subtitle": "Información del proyecto y beneficios",
  "bullets": ["Infraestructura", "Servicios", "Operación"],
  "contact": { "phone": "+57 ...", "email": "..." }
}
```

## Build & scripts mínimos
- `npm run preview` -> vista local
- `npm run pdf` -> genera PDF(s)
- `npm run lint` -> CSS/HTML
- `npm run validate` -> anti-overflow + assets missing
- Opcional: `npm run a11y` (Pa11y)
- Opcional: `npm run qa:visual` (screenshots + pixel diff)

Ejemplo `package.json`:
```json
{
  "type": "module",
  "devDependencies": {
    "playwright": "^1.0.0",
    "prettier": "^3.0.0",
    "stylelint": "^16.0.0",
    "stylelint-config-standard": "^36.0.0"
  },
  "scripts": {
    "pdf": "node packages/core/src/scripts/export-pdf.mjs",
    "lint": "stylelint \"**/*.css\"",
    "format": "prettier . --write"
  }
}
```

### Export PDF (Playwright) mejores prácticas
- `printBackground: true`
- márgenes consistentes
- `displayHeaderFooter` para numeración real (pageNumber/totalPages)
- PDFs por idioma o variante

## QA / validaciones
**Lint/validación HTML y CSS**
- html-validate (HTML offline)
- stylelint (CSS)
- `@html-eslint/eslint-plugin` si quieres ESLint sobre `.html`

**Accesibilidad (si los PDFs también viven como páginas web)**
- Pa11y (auditorías automatizables en CI)

**Visual regression (opcional, recomendado)**
- Playwright: renderizar páginas a PNG
- Comparar con baseline (pixelmatch o equivalente)

**Checks funcionales**
- No overflow: `scrollWidth > clientWidth`.
- Fuentes cargadas: fallar si faltan archivos.
- Imágenes faltantes: `img.naturalWidth === 0`.
- Saltos de página: elementos “must-not-break” completos.

## Post-procesado y mantenimiento de PDFs
- qpdf: re-structurar/linearizar (fast web view) y transformaciones.
- pdfcpu: optimización/validación/manipulación en CLI.
- Nota: la reducción de peso suele requerir decisiones sobre imágenes/fuentes; qpdf no siempre “comprime mágicamente”.

## Qué suele funcionar / qué suele fallar
**Funciona**
- Separar layout vs contenido (templates + content.json).
- Diseñar con mentalidad de imprenta (Paged Media, unidades físicas).
- Fuentes locales y assets versionados.
- Export reproducible por script en CI.

**Falla**
- Confiar en `window.print()` manual.
- Usar `wkhtmltopdf` para layouts modernos.
- No controlar saltos de página (títulos al final, cards partidas).
- Depender de recursos remotos sin “pin”.

## Flujo de trabajo para agentes (Codex)
**Contrato de edición**
- Cambio de texto -> editar solo `content/*.json`.
- Cambio de branding -> editar `tokens/*.json`.
- Cambio de layout -> editar template/CSS con cambios mínimos.
- Ejecutar siempre: `npm run pdf` y `npm run validate`.

**Prompt patrón**
Incluye:
- objetivo exacto (pieza, tamaño, páginas)
- restricciones (sin overflow, respetar grid, márgenes)
- archivos permitidos
- comandos de verificación
- definición de “done”

Ejemplo corto:
```
Actualiza projects/omnicity-brochure-2026/:
- Cambia el texto de “Qué hacemos” y agrega 2 bullets nuevos.
Reglas:
- Solo edita content/es.json (no tocar HTML/CSS).
Verifica:
- npm run pdf (debe generar PDF sin errores)
- npm run validate (sin overflow)
```

## Checklist final de “listo para imprimir / entregar”
- Tamaño correcto (A4/Letter) y márgenes definidos
- Sangrado (si aplica) y nada crítico cerca del borde
- Sin cortes feos (cards/títulos/listas no se parten)
- Imágenes nítidas, logos en SVG
- Fuentes embebidas o disponibles en el pipeline
- PDF con background y numeración si aplica
- Validación automática + (ideal) comparación visual

## Siguiente paso sugerido
Definir un caso real inicial (ej. “Brochure Omnicity 2 páginas A4 ES/EN”) y aterrizar:
- estructura exacta de carpetas
- templates
- tokens iniciales
- scripts `export-pdf` + `validate`
- prompts guardrail para Codex
