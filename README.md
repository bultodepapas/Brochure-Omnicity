# Brochure Omnicity

Repositorio para diseñar brochures y documentos informativos de Omnicity con un pipeline reproducible (HTML/CSS -> PDF), seguro para agentes y listo para impresión.

## Objetivo
Construir un flujo de trabajo profesional para piezas impresas y digitales, con separación clara entre layout y contenido, validaciones automáticas y exportación consistente de PDFs.

## Alcance actual
- Definir stack técnico.
- Establecer convenciones de impresión.
- Proponer estructura de repo y flujo de trabajo.
- Preparar el pipeline antes de producir piezas finales.

## Stack recomendado
**Decisión actual:** HTML/CSS + Playwright (Chromium print).

Por qué:
- Render idéntico a Chrome.
- Exportación a PDF confiable.
- Fácil automatización en CI.

Dependencias clave:
- `playwright`
- `prettier`
- `stylelint` (+ config)
- `eslint` (si hay JS)
- Opcional: `nunjucks`/`handlebars`/`mustache`
- Opcional: `vite`

## Mapa de herramientas (por capas)
- Autoría: HTML/CSS (VS Code)
- Templating: Nunjucks/Handlebars/Mustache
- Paginación: CSS Paged Media / Paged.js
- PDF: Playwright / WeasyPrint / PrinceXML / Antenna House
- QA: html-validate / stylelint / Pa11y / visual regression
- Optimización: qpdf / pdfcpu

## Herramientas complementarias (visual + programable)
### 1) “Figma-lite” dentro de VS Code
Para diagramas, wireframes y piezas rápidas versionadas en Git:
- **Excalidraw** (VS Code extension): edita `.excalidraw` (`.svg/.png/.json`) dentro de VS Code.
- **tldraw** (VS Code extension): whiteboard/infinite canvas con archivos `.tldr` offline.

Cuándo sirve: edición visual sin salir del workflow repo-driven (sin Figma).

### 2) Creación gráfica programática
**A) Creative coding**
- **p5.js** (+ ecosistema) para posters generativos, patrones y fondos.
- **p5.plotSvg** si necesitas exportar SVG “plotter friendly”.

**B) Vector con API limpia**
- **Paper.js** (scene graph sobre Canvas) para shapes, curvas y operaciones tipo Illustrator.

**C) Editor gráfico embebido**
- **Fabric.js** (canvas con objetos + parse SVG).
- **Konva** (stage/layers/shapes con eventos).

### 3) SVG como formato fuente
**Manipulación/animación**
- **SVG.js** para manipular y animar SVG sin dependencias.

**Optimización**
- **SVGO** (CLI + librería) para limpiar y reducir tamaño.
- **SVGOMG** (GUI de SVGO) para ajustes manuales.

### 4) Tipografía programática
- **opentype.js** para inspeccionar glifos, kerning y convertir texto a paths.

Nota: al renderizar SVG -> PNG/PDF, el texto puede romperse si no controlas fuentes; considera convertir a paths cuando sea crítico.

### 5) Render/export industrial (HTML/SVG -> PNG/PDF)
**Generación de SVG desde layout tipo HTML/CSS**
- **Satori** (npm) para templates repetibles (cards, hero, etc.).

**Rasterización de SVG**
- **resvg-js / resvg-wasm** (renderer rápido y consistente).

**Procesamiento de imágenes**
- **sharp** para resize/optimización/formatos (PNG/JPEG/WebP/AVIF).

Gotcha: Satori -> SVG -> sharp puede fallar en algunos casos; mucha gente usa **resvg** como renderer final.

### 6) Paginación editorial avanzada
- **Paged.js** (paginación print-ready en navegador).
- **Vivliostyle** (Viewer/CLI con salida PDF; opción PDF/X-1a).

### 7) Animación y gráficos “de brochure”
- **GSAP** (SVG/DOM/canvas).
- **Motion** (alternativa moderna, buen rendimiento).
- **Lottie-web** (animaciones AE/Bodymovin en web).

### 8) Color “como diseñador” en código
- **Culori** (OKLCH, escalas y paletas).
- **OKLCH.com** (picker/conversor práctico).

### Toolbox mínimo recomendado
Para un flujo gráfico + programable + exportable sin complejidad excesiva:
1. Fuente: **SVG** como assets + **HTML/CSS** como layout.
2. Edición visual: **Excalidraw** o **tldraw** dentro de VS Code.
3. Generación: fondos/patrones con **p5.js** o **Paper.js**; composiciones UI con **Satori -> SVG**.
4. Render final: **resvg** (SVG -> PNG) + **sharp** (optimización).
5. Higiene: **SVGO** para limpiar SVGs.
6. Si es multi‑página: **Vivliostyle** o **Paged.js**.

## Estructura de repo (propuesta)
```
print-studio/
  packages/
    core/
      src/
        tokens/
        styles/
        templates/
        scripts/
  projects/
    omnicity-brochure-2026/
      content/
      assets/
      src/
      dist/
      project.config.json
  .github/workflows/
    pdf.yml
```

## Convenciones de impresión (no negociables)
- `@page` con tamaño y márgenes definidos.
- `break-inside: avoid` y `break-before: page` en elementos críticos.
- Usar unidades físicas: `mm`, `pt`, `cm`.
- Fuentes locales en `assets/fonts` con `@font-face`.
- Logos/íconos en SVG.

## Motor de contenido
- **Tokens** en `tokens/*.json` como fuente de verdad.
- **Contenido** en `content/*.json` para que los agentes editen texto sin romper el layout.

## Flujo de trabajo para agentes (Codex)
Contrato de edición:
- Texto -> editar solo `content/*.json`.
- Branding -> editar `tokens/*.json`.
- Layout -> editar templates/CSS con cambios mínimos.
- Verificación obligatoria: `npm run pdf` y `npm run validate`.

## QA mínimo recomendado
- Overflow: `scrollWidth > clientWidth`.
- Imágenes faltantes: `img.naturalWidth === 0`.
- Fuentes cargadas: archivos presentes.
- Visual regression (opcional): PNGs + pixel diff.

## Comandos (previstos)
- `npm run preview`
- `npm run pdf`
- `npm run lint`
- `npm run validate`

## Documentos de referencia
- `base.md` (guía completa de stack, QA y prácticas)

## Próximo paso
Definir un caso real inicial (por ejemplo: “Brochure Omnicity 2 páginas A4 ES/EN”) y aterrizar:
- estructura exacta de carpetas
- templates
- tokens iniciales
- scripts `export-pdf` + `validate`
- prompts guardrail para Codex
