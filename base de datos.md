# OmniCity (Omnicity)
Plataforma digital integral para la gestión moderna de ciudades. Conecta ciudadanía, administración y territorio en un solo ecosistema.

## Metadatos del documento
- Versión del documento: 2.3
- Fecha y hora de última actualización: 2026-01-29 11:48:04 (hora local del repositorio)
- Alcance: Documento maestro para inversionistas y brochure comercial para alcaldías
- Fuente principal: Repositorio D:\DEVELOMENT\omnicity-1 (código, docs y guías internas)
- Versión de la app (pubspec): 1.0.0+1

---

## Resumen ejecutivo
OmniCity (Omnicity) es una plataforma multiciudad (multi-tenant) y de marca blanca (white-label) que permite a un municipio operar su ciudad digital desde un único ecosistema: app móvil ciudadana (Android/iOS), web/PWA y panel administrativo, sobre un backend seguro con datos centralizados. Su arquitectura modular habilita o desactiva servicios por ciudad sin desarrollar nuevas apps, y personaliza la experiencia mediante configuración dinámica, branding y feature flags.

La plataforma integra comunicación oficial, trámites digitales, reportes ciudadanos geolocalizados, servicios urbanos y economía local (empleos, clasificados, turismo, mascotas), con analítica basada en datos reales. El resultado: menos fragmentación institucional, mayor trazabilidad, más participación ciudadana y mejor transparencia.

---

## 1. Problema que resuelve
Las administraciones municipales enfrentan desafíos estructurales:
- Información institucional fragmentada y dispersa.
- Canales de atención desconectados (WhatsApp, redes sociales, ventanillas físicas).
- Baja trazabilidad y seguimiento de solicitudes ciudadanas.
- Dificultad para comunicar acciones de gobierno y alertas urgentes.
- Ausencia de datos unificados para decisiones basadas en evidencia.
- Baja apropiación tecnológica por parte de la ciudadanía.

**OmniCity centraliza, ordena y digitaliza la gestión municipal**, convirtiendo el celular del ciudadano en el canal oficial principal y más confiable del municipio.

---

## 2. Qué es OmniCity (en términos simples)
- Una app móvil oficial del municipio (Android/iOS) con identidad gráfica propia.
- Una web/PWA ciudadana y un panel administrativo para gestionar contenidos y procesos.
- Un backend centralizado (Supabase) con seguridad y aislamiento por ciudad.
- Un sistema modular, escalable y configurable por necesidad territorial.
- Un canal directo, verificable y medible entre gobierno local y ciudadanía.

---

## 3. Componentes de la plataforma

### 3.1 App móvil ciudadana
- Disponible para Android e iOS.
- Interfaz dinámica (Home) configurable desde backend.
- Módulos activables por ciudad (feature flags).
- Experiencia optimizada para uso diario y en baja conectividad.

### 3.2 Web / PWA
- Despliegue web por ciudad (subdominio propio).
- Carga optimizada con deferred loading y pantalla de carga personalizada.
- Compatible con instalación como PWA (según configuración del tenant).

### 3.3 Panel administrativo web
- Construido en el mismo codebase Flutter (modo Web).
- Gestión de cuestionarios y formularios dinámicos.
- Acceso por roles (Admin / Editor).
- Roadmap de dashboard de reportes y analítica avanzada.

### 3.4 Backend y datos
- Supabase (PostgreSQL + Auth + Storage + RLS + Realtime).
- Aislamiento multi-tenant por `tenant_id` y políticas RLS.
- Edge Functions activas: eliminación de cuenta (`delete_account`) y pipeline financiero (`finance_ingest`).

---

## 4. Arquitectura y principios

### 4.1 Arquitectura por módulos (Feature-First + Clean Architecture)
Cada feature es un módulo autocontenido con capas **Domain / Data / Presentation**, lo que facilita escalabilidad, mantenibilidad y pruebas.

### 4.2 Multi-tenancy (white-label real)
- Cada ciudad se compila con un `TENANT_ID` propio.
- Branding, módulos visibles y layout del Home se cargan desde backend.
- Una sola base de código sirve a múltiples municipios.

### 4.3 Home dinámico con motor Tetris
El Home se renderiza desde una configuración JSON (layout dinámico) y organiza los módulos con un motor de grilla adaptable por dispositivo, permitiendo priorizar información sin publicar nuevas versiones en tiendas.

### 4.4 UI guiada por servidor en Trámites
Los trámites se renderizan desde bundles JSON. Esto permite crear o modificar flujos administrativos sin cambiar la app, reduciendo tiempos de implementación.

### 4.5 Offline-first y resiliencia
- Caché local con Hive (ej: alertas, turismo, religión).
- Estrategia de fallback en datos críticos (ej: finanzas).
- Persistencia segura de sesiones (Secure Storage en móvil, localStorage en web).

### 4.6 Stack tecnológico y versiones base
- Flutter 3.x + Dart SDK 3.10.3.
- Supabase Flutter SDK 2.12.0 (Auth, DB, Storage, Realtime).
- Riverpod 3.1.0 (state management).
- GoRouter 17.0.1 (navegación y rutas tipadas).
- flutter_map 8.2.2 (mapas) + geolocator 14.0.2 (ubicación).
- Hive 2.2.3 + Flutter Secure Storage 10.0.0 (offline y seguridad).
- Internacionalización base en español (es_CO).

---

## 5. Seguridad, gobernanza y cumplimiento
- Autenticación con Supabase (Email/Password, Google, Facebook).
- Políticas RLS estrictas para aislar datos por usuario y por tenant.
- Almacenamiento seguro de sesiones y cifrado local.
- Acceso por roles para administradores y editores.
- Auditoría y políticas de almacenamiento definidas en `shared/contracts/policies`.

---

## 6. Módulos funcionales (qué hace y cómo lo hace)

### 6.1 Core e Identidad
**Splash**
- Inicio inteligente: carga branding por ciudad, valida versión mínima y define ruta (home o auth).
- Precarga colores y activos para reducir el tiempo percibido de inicio.

**Auth**
- Registro e inicio de sesión con Email/Password, Google y Facebook.
- Recuperación de contraseña y sesión persistente y segura (móvil y web).

**User (Perfil Ciudadano)**
- Perfil con datos personales, avatar y preferencias.
- Asociación a barrio y configuración de notificaciones por categoría.
- Puntos y estadísticas de participación (base para gamificación).

**Cities / Tenants**
- Configuración por ciudad: branding, módulos habilitados y parámetros locales.
- Feature flags para activar o apagar módulos sin nuevas publicaciones.
- Resolución de assets y mensajes por ciudad.

**Home dinámico**
- Pantalla principal configurable desde backend (layout JSON).
- Motor de grilla adaptable según dispositivo.
- Carruseles, accesos rápidos y módulos destacados según prioridades.

---

### 6.2 Comunicación oficial
**Alertas (Alerts)**
- Alertas push con severidad y categoría.
- Historial oficial persistente + caché local para consulta offline.
- Segmentación por ciudad, barrio o grupo objetivo.

**Noticias (News)**
- Feed oficial con artículos, imágenes y prioridad editorial.
- Integración directa al Home para máxima visibilidad.
- Lectura clara y controlada (fuente oficial del municipio).

**Eventos (Events)**
- Calendario con filtros por categoría, fechas y gratuidad.
- Detalle con ubicación, mapa y botón “Agregar al calendario”.
- Creación desde app ciudadana desactivada (ver limitaciones).

---

### 6.3 Participación ciudadana y trámites
**Reportes Ciudadanos (Reports / PQRS)**
- Reporte guiado con foto, ubicación GPS y categoría.
- Imágenes comprimidas y almacenadas en Supabase Storage.
- Estado del caso (pendiente, en progreso, resuelto, rechazado) con trazabilidad.
- Mapa de reportes y validación social para priorizar intervención.

**Trámites (Procedimientos Digitales)**
- UI guiada por servidor: la app interpreta bundles JSON con pasos, campos y reglas.
- Cambios de flujo sin publicar nueva versión en tiendas.
- Motor de reglas (JsonLogic) para condiciones y validaciones.

**Encuestas (Surveys)**
- Votaciones rápidas con resultados en tiempo real.
- Control anti doble voto y segmentación demográfica (edad, barrio, género).
- Insumo para priorizar políticas y proyectos públicos.

**Cuestionarios (Questionnaire)**
- Constructor web con versionado y control de publicación.
- Render móvil para diligenciamiento ciudadano o en campo.
- Respuestas almacenadas y analizables para decisiones.

---

### 6.4 Servicios urbanos y utilidad pública
**Emergencias (Emergency)**
- Botón SOS con marcación inmediata del número principal de emergencia.
- Directorio completo de contactos críticos (policía, bomberos, salud, servicios públicos).
- Cuadrantes policiales según barrio del usuario.
- Funciona sin conexión (offline-first).

**Medio Ambiente (Environment)**
- Calidad del aire (AQI), niveles de río y horarios de recolección de basura.
- Datos segmentados por barrio y fuentes IoT/externas según ciudad.
- Variables complementarias cuando están disponibles (temperatura, humedad, UV).

**Finanzas (Finance)**
- Indicadores económicos locales (TRM, commodities, etc.).
- Pipeline con caché y fallback (local -> Supabase -> externo -> último valor).
- Ingesta programada para mantener datos actualizados.

**Links Oficiales (Links)**
- Directorio curado de enlaces oficiales (pagos, trámites externos, servicios).
- Categorías, iconografía y promoción de links críticos.
- Apertura segura dentro de la app para no perder contexto.

---

### 6.5 Comunidad y economía local
**Empleos (Jobs)**
- Bolsa local con filtros por sector y ubicación.
- Contacto rápido (WhatsApp/llamada) y verificación de empleadores.
- Expiración automática para mantener ofertas vigentes.

**Clasificados (Classifieds)**
- Publicaciones por categoría con contacto directo (teléfono/WhatsApp).
- Flujo completo: crear, listar, gestionar publicaciones propias y vencimiento.
- Estado de publicación para control del mercado local.

**Mascotas (Pets)**
- Modo perdidos/encontrados y adopción con foto, ubicación y contacto.
- Estados del caso y priorización por urgencia.
- Contacto visible solo a usuarios autenticados para evitar scraping.

**Turismo (Tourism)**
- Catálogo de atractivos con categorías, tags, horarios y prioridad editorial.
- Listas filtradas, destacados y soporte de mapas.
- Contenido cacheado para baja conectividad.

**Religión**
- Tipos de culto configurables en base de datos (sin código).
- Directorio de organizaciones religiosas y filtros por preferencia.
- Administración por roles para contenidos religiosos.

**Barrios / Neighborhoods**
- Base geográfica unificada: barrios, zonas y comunas.
- Segmenta alertas, reportes, recolección de basura y encuestas.
- Conecta la analítica territorial con la operación municipal.

**Sistema de niveles de vecino (gamificación)**
- Nivel ciudadano calculado por actividad (reportes, mascotas, empleos).
- Actualización en tiempo real.
- Incentiva participación y permanencia en la plataforma.

---

## 7. Analítica y datos
OmniCity genera datos accionables para la toma de decisiones:
- Volumen y tipo de reportes por barrio.
- Tendencias de participación (encuestas y cuestionarios).
- Eventos y noticias con mayor impacto.
- Alertas críticas y zonas de riesgo.
- Indicadores ambientales por zona.

Estos datos permiten decisiones basadas en evidencia y priorización eficiente de recursos.

---

## 8. Implementación por ciudad (onboarding de nuevo municipio)
Proceso estandarizado para habilitar un nuevo tenant:
1. Crear tenant en Supabase (`tenants`) con nombre y config base.
2. Cargar branding (logos, colores, fondos de login, barra superior, saludos).
3. Configurar layout del Home (`city_layout_config`).
4. Activar feature flags por módulo (`tenant_feature_flags`).
5. Cargar contenido mínimo por módulo (news, events, tourism, trámites, links).
6. Publicar app en stores con identidad propia.
7. Desplegar web/PWA en Vercel con subdominio propio y variables de entorno.

---

## 9. Despliegue y operaciones
- Compilación por ciudad con `TENANT_ID` y variables de entorno.
- Deploy web en Vercel con build script `vercel_build.sh`.
- Soporte de múltiples proyectos Vercel, uno por ciudad.
- Seguridad de sesiones por plataforma (Secure Storage en móvil, localStorage en web).

---

## 10. Limitaciones y funciones desactivadas (para transparencia)
- **Creación de eventos desde la app ciudadana:** desactivada desde 2026-01-06.
- **Dashboard admin de reportes:** en roadmap (no disponible en UI actual).
- Algunos módulos tienen backlog de mejoras (ver sección 11).

---

## 11. Roadmap y backlog (resumen de mejoras identificadas)
*(No implementadas al momento, sujetas a priorización)*
- Alertas: rich media, confirmación “Estoy a salvo”, recibos de lectura.
- Emergencias: envío de ubicación, mapa offline de hospitales/policía.
- Reportes: gamificación avanzada, timeline de estados, análisis automático de imagen.
- Trámites: guardado de borradores, carga de archivos, pasarelas de pago.
- Encuestas: visualizaciones avanzadas y drafts.
- Turismo: geolocalización avanzada, mapas y galería de imágenes.
- Home: personalización por usuario y widgets en vivo.

---

## 12. Diferenciadores clave
- Multi-tenant real: una sola tecnología, múltiples ciudades.
- Home dinámico configurable sin actualizaciones en tiendas.
- Trámites server-driven (reduce tiempo de lanzamiento de servicios).
- Offline-first en módulos críticos.
- Seguridad y aislamiento de datos con RLS.
- Diseño modular y escalable para crecimiento regional.

---

## 13. Propuesta de valor para alcaldías
- Canal oficial directo y verificable con la ciudadanía.
- Mayor trazabilidad y control sobre solicitudes ciudadanas.
- Datos en tiempo real para decisiones públicas.
- Reducción de costos administrativos y tiempos de atención.
- Legado digital sostenible para futuras administraciones.

---

## 14. Glosario rápido
- **Tenant**: municipio/ciudad con configuración propia.
- **Feature Flag**: interruptor para activar o desactivar módulos.
- **UI guiada por servidor (Server-Driven UI)**: pantallas definidas por backend (sin update en app).
- **RLS**: Row Level Security, control de acceso por fila en base de datos.

---

**Fin del documento.**
