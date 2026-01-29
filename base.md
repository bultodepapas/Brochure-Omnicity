Stack recomendado (robusto y “agent-friendly”)
Opción A (recomendada): Node + Playwright (Chromium print)

Renderiza HTML/CSS como lo hace Chrome y lo imprime a PDF.

Excelente para layouts modernos, fuentes web, grids, etc.

Muy estable para automatizar en CI.

Dependencias clave

playwright (export PDF)

prettier (formato)

stylelint (+ config) (higiene CSS)

eslint (si usas JS para plantillas)

(opcional) vite (preview rápido / bundling de assets)

(opcional) mustache/nunjucks/handlebars para templating

Opción B: Python + WeasyPrint

Muy buena para documentos “editoriales”.

A veces menos compatible con CSS moderno que Chromium.


ejemplo de estructura: Estructura de repo (multi-pieza, escalable)
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
          poster.html
        scripts/
          render.mjs
          export-pdf.mjs
          validate.mjs
      package.json
  projects/
    servin-brochure-2026/
      content/
        es.json
        en.json
      assets/
        images/
        fonts/
        logos/
      src/
        index.html            # o entry template
        styles.css            # overrides del proyecto
      dist/
        servin-brochure-es.pdf
        servin-brochure-en.pdf
      project.config.json     # tamaño, bleed, etc.
  .github/workflows/
    pdf.yml
  README.md


  Convenciones de diseño para impresión (no negociables)
4.1 Print CSS

Define hoja/márgenes con @page.

Evita cortes feos con:

break-inside: avoid;

break-before: page;

Usa unidades “físicas”:

mm, pt, cm para spacing y tipografía en impresión.

Ejemplo base:

@page { size: A4; margin: 14mm; }
* { box-sizing: border-box; }
body { font: 11pt/1.45 system-ui; }
.break { break-before: page; }
.card, h2, ul { break-inside: avoid; }

4.2 Tipografía

Preferible fuentes locales dentro del repo (assets/fonts) y declaradas con @font-face.

Define escala tipográfica (ej: 28pt, 16pt, 11pt, 9pt) y no improvises.

4.3 Imágenes

Logos/íconos en SVG siempre que puedas.

Fotos a 300 DPI equivalente (depende del tamaño impreso).

Mantén un assets/ organizado y nombres consistentes.

4.4 Color (impresión real)

La web es RGB; la imprenta puede pedir CMYK/PDF/X.
En la práctica: exporta buen PDF desde Chromium y, si vas a imprenta profesional, valida requisitos (perfil, sangrado, marcas) y ajusta el pipeline (a veces hay paso adicional con herramientas de prepress).

Para la mayoría de brochures “corporativos” digitales/impresión rápida: Chromium PDF + sangrado correcto suele ser suficiente.

5) “Motor” de contenido: tokens + contenido (para que el agente no rompa el layout)
5.1 Design Tokens (fuente de verdad)

brand.tokens.json:

{
  "colors": { "ink": "#111111", "muted": "#666666", "border": "#E5E5E5" },
  "spacingMm": { "xs": 2, "sm": 4, "md": 6, "lg": 10 },
  "radiiMm": { "sm": 2, "md": 3 },
  "typePt": { "h1": 28, "h2": 14, "body": 11, "small": 9 }
}


Los scripts convierten tokens → variables CSS:

:root{
  --ink:#111;
  --space-md:6mm;
  --h1:28pt;
}

5.2 Contenido en JSON (editable por agents sin tocar layout)

content/es.json:

{
  "title": "SERVIN 2026",
  "subtitle": "Servicios industriales con foco en seguridad",
  "bullets": ["Soldadura", "Mantenimiento", "Automatización"],
  "contact": { "phone": "+57 ...", "email": "..." }
}


La plantilla (Nunjucks/Handlebars o HTML con placeholders) lo inserta.

6) Build & scripts (lo que hace “producto” al repo)
Scripts mínimos

npm run preview → levanta vista local

npm run pdf → genera PDF(s)

npm run lint → CSS/JS

npm run validate → checks anti-overflow + tamaño + assets missing

Ejemplo package.json (core o raíz):

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

Export PDF (Playwright) mejores prácticas

printBackground: true

margin consistente

displayHeaderFooter para numeración real (pageNumber/totalPages)

Genera PDFs por idioma o variante.

7) QA: cómo asegurar que “no se dañó el brochure”

Esto es clave cuando un agente edita.

Validaciones automáticas útiles

No overflow: en Playwright, evalúas si algún contenedor se sale del ancho/alto esperado (scrollWidth > clientWidth).

Fuentes cargadas: falla si faltan archivos de fuentes.

Imágenes faltantes: detecta img.naturalWidth === 0.

Saltos de página: valida que elementos “must-not-break” estén completos.

Visual regression (opcional pero oro)

Renderizas cada página a PNG y comparas contra baseline (pixelmatch).

Si hay diferencia > umbral, CI falla.

Esto hace que los agents trabajen “seguros”: pueden iterar sin romper producción.

8) Flujo de trabajo para AI agents (Codex) que realmente funciona
Contrato de edición (reglas)

Si el cambio es texto → editar content/*.json únicamente.

Si es branding → editar tokens/*.json.

Si es layout → editar template/CSS con cambios mínimos.

Siempre ejecutar: npm run pdf y npm run validate.

Prompt patrón (súper efectivo)

Incluye:

objetivo exacto (tipo de pieza, tamaño, páginas)

restricciones (sin overflow, respetar grid, márgenes)

archivos permitidos a tocar

comandos de verificación

definición de “done”

Ejemplo corto:

Actualiza projects/servin-brochure-2026/:
- Cambia el texto de “Qué hacemos” y agrega 2 bullets nuevos.
Reglas:
- Solo edita content/es.json (no tocar HTML/CSS).
Verifica:
- npm run pdf (debe generar PDF sin errores)
- npm run validate (sin overflow)

9) Checklist final de “listo para imprimir / entregar”

 Tamaño correcto (A4/Letter) y márgenes definidos

 Sangrado (si aplica) y nada crítico cerca del borde

 Sin cortes feos (cards/títulos/listas no se parten)

 Imágenes nítidas, logos en SVG

 Fuentes embebidas o disponibles en el pipeline

 PDF generado con background y numeración si aplica

 Validación automática + (ideal) comparación visual

Si quieres, te lo dejo aterrizado a un caso real (por ejemplo “SERVIN brochure 2 páginas A4 ES/EN”) y te propongo:

estructura exacta de carpetas,

templates,

tokens iniciales,

scripts export-pdf + validate,

y un set de prompts “guardrail” para Codex.
Sí — hay bastante gente haciendo “design docs/brochures en HTML/CSS en un repo y exportando a PDF”. Al revisar patrones repetidos (y dónde se rompen), esto es lo que mejor funciona y lo que suele salir mal.

1) Qué hace la gente que le sale “pro”
A. HTML/CSS + motor de paginación (CSS Paged Media)

Los workflows más “print-first” se apoyan en CSS Paged Media (@page, regiones, saltos, contadores) y un renderer que lo respete bien. PrinceXML documenta muy claramente este modelo (tamaño, regiones, headers/footers, paginación).

En open source, Paged.js es un estándar de facto para paginar HTML “como libro/revista” en el navegador y luego imprimir/exportar.

Para pipelines Python, WeasyPrint se usa muchísimo para HTML→PDF con CSS paginado y es común ver plantillas + “templating” (Jinja/EJS) para contenido.

B. HTML + “print to PDF” con Chromium (Playwright/Puppeteer)

Mucha gente usa Playwright porque imprime lo que el navegador renderiza (incluye JS, fuentes web, etc.) y permite headerTemplate/footerTemplate, márgenes y printBackground.

2) Lo que funciona (patrones ganadores)

Separar “layout” vs “contenido”

Layout estable en src/template.html + styles.css.

Contenido en content.json|yaml|md para que los agents editen texto sin romper el diseño. (Esto aparece mucho en ejemplos con WeasyPrint/Jinja o HTML→PDF con templates).

Diseñar con mentalidad de imprenta (no de web)

Usar @page, tamaños (A4/Letter), márgenes en mm, reglas de salto (break-before, break-inside), y evitar elementos que queden “partidos”. Es la base de CSS para impresión y paged media.

Tipografía y assets deterministas

Fuentes locales en repo (assets/fonts) + @font-face.

SVG para logos/iconos.

Esto reduce el clásico “en mi máquina se ve distinto”.

Export reproducible por script (CI-friendly)

Un comando único: npm run pdf (Playwright) o make pdf (WeasyPrint/Prince/Paged.js).

Commit del PDF final solo si lo necesitas; si no, que sea artifact de CI.

3) Lo que NO funciona (fallos típicos)

Confiar en window.print() manual

Se vuelve inconsistente entre máquinas/OS, y los márgenes/headers cambian.

Usar wkhtmltopdf para layouts modernos

Sigue apareciendo en comparativas, pero suele quedarse atrás con CSS moderno (flex/grid, fuentes, etc.) vs motores basados en Chromium (Playwright/Puppeteer).

No controlar saltos de página

El resultado “amateur” casi siempre es: títulos al final de una página, cards partidas, viudas/huérfanas, bullets cortados. (Paged Media y/o Paged.js/Prince resuelven esto mejor).

Depender de recursos remotos sin “pin”

Fuentes desde CDN sin lock → un día cambian y el PDF “reflowea”.

4) Recomendación de stack para tu caso (VS Code + Codex agents)

Te dejo 3 stacks, de más simple a más editorial:

Stack 1 — “Simple y potente”: Playwright (Chromium) + HTML/CSS

Ideal: 1–4 páginas tipo brochure corporativo, fichas, one-pagers.

Pros: fiel al navegador, fácil en Node, headers/footers con templates.

Contras: paginación avanzada (running headers complejos, floats editoriales) cuesta más que con Prince/Paged.js.

Stack 2 — “Editorial open source”: Paged.js + Playwright

Ideal: documentos largos, estilo revista/libro, control fino de paginación.

Pros: Paged.js aporta paginación “de verdad” basada en CSS paged media.

Contras: más moving parts (JS de paginación).

Stack 3 — “Imprenta pro”: PrinceXML (pago) / Antenna House (enterprise)

Ideal: calidad de imprenta, features avanzados de paged media y control extremo.

5) Mejores prácticas de codebase (para que agents trabajen bien)

Estructura recomendada

brochures/<pieza>/src/template.html

brochures/<pieza>/src/styles.css

brochures/<pieza>/src/content.json

brochures/<pieza>/assets/{images,fonts}

brochures/<pieza>/scripts/export-pdf.(mjs|py)

brochures/<pieza>/dist/<pieza>.pdf

Reglas de oro

Todo spacing importante en mm (no px).

Usar una grilla fija (12 col) y tokens: --space-1..n, --font-s..xl.

CSS “print-only”: @media print { … } + @page { … }.

Tests visuales básicos: generar PDF y revisar que no haya overflow/cortes (en CI si puedes).

6) “Qué pedirle a Codex” para evitar que destruya el layout

Tu prompt al agente debería imponer:

No tocar la grilla/tokens sin necesidad.

Cambios de texto solo en content.json.

Validar: “no overflow”, “no cortes feos”, “A4”, “printBackground”.

Ejecutar npm run pdf y confirmar output.