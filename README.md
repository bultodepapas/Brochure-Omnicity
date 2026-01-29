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
