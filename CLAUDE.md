# Mecatos el Santi — ERP/POS Multisede

Contexto de proyecto para agentes de IA (Claude Code, Cursor, Copilot, etc.).
Léelo antes de tocar código — resume decisiones de arquitectura y errores ya
resueltos que no deberían repetirse.

## Qué es esto

ERP/POS multisede para una cadena de comidas en Colombia. Monorepo con:

```
mecatos-erp/
├── backend/            API REST (Express) + worker de colas (BullMQ/Redis)
│                         ver backend/CLAUDE.md
├── admin-frontend/      App ÚNICA: login por rol, panel admin y zona de cajero
│                         ver admin-frontend/CLAUDE.md
│   └── src/cajero/      Zona de cajero (Caja/Facturas/Compras)
│                         ver admin-frontend/src/cajero/CLAUDE.md
├── pos-frontend/        ⚠️ LEGADO — ver pos-frontend/README.md, no lo edites
│                         para features nuevas salvo que se te pida explícitamente
└── print-server/        Servicio local de impresión térmica (ESC/POS) — corre
                          en la terminal física con la impresora USB, NO en la
                          nube; ver print-server/CLAUDE.md y docs/
                          THERMAL_PRINTER_INTEGRATION.md
```

**Cambio de arquitectura importante (léelo antes de tocar frontend):**
originalmente había dos apps separadas (`pos-frontend` para cajeros,
`admin-frontend` para administradores). Ahora **todo vive en una sola app**
(`admin-frontend`), con una pantalla de login que diferencia el rol y manda
a cada quien a su zona:

- Admin/Manager → `/dashboard` y el resto de rutas RBAC ya existentes.
- Cajero → `/cajero/*`, un área con 3 pestañas (Caja / Facturas / Compras)
  definida en `admin-frontend/src/cajero/` (módulo separado dentro de la
  misma app, con su propio store, cliente API y token).

`pos-frontend` sigue en el repo pero es legado — el código real de "Caja"
vive ahora en `admin-frontend/src/cajero/`, que es una copia evolucionada
(no un symlink ni un import cross-package). **Si haces un cambio al flujo de
venta, hazlo en `admin-frontend/src/cajero/`, no en `pos-frontend/`** — son
código duplicado a propósito tras la migración, no sincronizado
automáticamente.

Cada paquete (`backend`, `admin-frontend`, `pos-frontend`, `print-server`)
tiene su propio `package.json` y se instala/corre por separado. No hay
workspaces de npm/yarn — comparten convenciones, no dependencias ni
node_modules.

## Cómo está organizado este contexto

Este archivo raíz es lo único que se carga siempre — solo tiene lo que
aplica a **todo** el proyecto (modelo de auth, zona horaria, convenciones
generales). El detalle de cada paquete vive en su propio `CLAUDE.md`:

- `backend/CLAUDE.md` — API, workers, colas, modelos, endpoints.
- `admin-frontend/CLAUDE.md` — panel admin, más las reglas compartidas con
  la zona de cajero (se heredan automáticamente si trabajas dentro de
  `src/cajero/`).
- `admin-frontend/src/cajero/CLAUDE.md` — solo lo específico de la zona de
  cajero (hereda todo lo de `admin-frontend/CLAUDE.md`).
- `print-server/CLAUDE.md` — impresión térmica.

**Documentación para usuarios finales (no es contexto de código):**
`docs/USER_MANUAL.md` es la **guía operativa oficial** para cajeros, gerentes
y administradores (login, Caja y ventas, pedidos DiDi, turnos, Dashboard,
conciliación DiDi de los miércoles, inventario/Carga Masiva y solución de
problemas frecuentes). Está escrita en español no técnico, con placeholders
de capturas en `docs/assets/manual/*.png` (las imágenes reales aún hay que
tomarlas). **Si cambias un nombre de botón, pantalla o flujo que ese manual
describe, actualízalo en el mismo cambio** — usa los textos exactos de la
UI, no los nombres internos del código (ej. el botón es "Carga Masiva" y el
modal "Asignación Masiva de Inventario"; la conciliación DiDi vive en
`Ventas`, no en Cuentas por Cobrar, que es una pantalla aparte de créditos
manuales). No lo cargues para trabajo de código: no describe arquitectura.

**A propósito este archivo NO importa esos archivos** (sin `@backend/
CLAUDE.md` etc.) — la idea de dividir el documento es justo que una sesión
o subagente que solo toca `print-server/` no cargue de una las 900 líneas
de backend/admin/cajero, y viceversa. Si vas a lanzar un subagente
orquestado a trabajar en un paquete específico, dale ese `CLAUDE.md` (más
este raíz) en vez de todo el árbol.

Los puntos siguen numerados igual que antes de dividir el archivo (no se
renumeraron) — el código tiene decenas de comentarios tipo "ver punto 19
de CLAUDE.md" que asumen esa numeración. Este índice te dice en qué
archivo vive cada número:

| Punto | Tema | Archivo |
|---|---|---|
| 1 | `asyncHandler` obligatorio en rutas async | `backend/CLAUDE.md` |
| 2 | Emisión DIAN 100% async vía BullMQ | `backend/CLAUDE.md` |
| 3 | Job de reconciliación DIAN | `backend/CLAUDE.md` |
| 4 | Redis: `REDIS_URL` vs host/puerto, gotcha `redis://`/`rediss://` | `backend/CLAUDE.md` |
| 5 | Worker DIAN en proceso separado | `backend/CLAUDE.md` |
| 6 | Auth dual (JWT admin/manager, sesión PIN cajero) | *(este archivo)* |
| 7 | `jwt.sign` y tipos de `expiresIn` | `backend/CLAUDE.md` |
| 8 | POS ya no crea ventas offline | `admin-frontend/src/cajero/CLAUDE.md` |
| 9 | Modals de Caja vía Zustand + `CustomEvent` | `admin-frontend/src/cajero/CLAUDE.md` |
| 10 | `dianService.ts` único punto de integración con el PTA | `backend/CLAUDE.md` |
| 11 | Helper único de procesamiento de imágenes | `admin-frontend/CLAUDE.md` |
| 12 | `src/cajero/` es una app dentro de la app | `admin-frontend/src/cajero/CLAUDE.md` |
| 13 | Cookies httpOnly, no localStorage, no fetch manual | *(este archivo)* |
| 14 | SKU de producto generado en el backend | `backend/CLAUDE.md` |
| 15 | Paginación desde el backend en endpoints `list*` | `backend/CLAUDE.md` |
| 16 | Compras: cajero y admin, ligadas a inventario | `admin-frontend/CLAUDE.md` |
| 17 | Formularios de admin: modal dedicado, no inline | `admin-frontend/CLAUDE.md` |
| 18 | ADMIN sin sede, MANAGER restringido a la suya, `resolveBranchFilter` | `backend/CLAUDE.md` |
| 19 | Ambos flujos de venta validan/descuentan stock | `admin-frontend/CLAUDE.md` |
| 20 | Restricciones de MANAGER (Sedes/Personal/Compras) | `admin-frontend/CLAUDE.md` |
| 21 | Sincronización con Google Sheets (cola + worker) | `backend/CLAUDE.md` |
| 22 | `ActionsMenu.tsx`, soft-cancel de ventas, filtros de Ventas | `admin-frontend/CLAUDE.md` |
| 23 | Finanzas (Caja + Reportes exportables) | `admin-frontend/CLAUDE.md` |
| 24 | `dateRange.ts` — inicio/fin de día en Bogotá | `backend/CLAUDE.md` |
| 25 | Dashboard (`/dashboard`) | `admin-frontend/CLAUDE.md` |
| 26 | Admin no puede autodesactivarse; autoeditarse lo desloguea | `admin-frontend/CLAUDE.md` |
| 27 | Toggle claro/oscuro de `Login.tsx` | `admin-frontend/CLAUDE.md` |
| 28 | Modal `fixed inset-0` necesita `!m-0` | `admin-frontend/CLAUDE.md` |
| 29 | Apertura/cierre de turno con verificación de stock | `admin-frontend/src/cajero/CLAUDE.md` |
| 30 | Producto ya en la orden se deshabilita en el grid | `admin-frontend/src/cajero/CLAUDE.md` |
| 31 | Sin turno abierto: aviso, nunca atrapado | `admin-frontend/src/cajero/CLAUDE.md` |
| 32 | Impresión térmica (`print-server/`) | `print-server/CLAUDE.md` |
| 33 | Zona horaria de Colombia en todo el proyecto | *(este archivo)* |
| 34 | Ventas Rappi/DiDi (canal, no método de pago) nacen `PENDING_PAYMENT` bajo `BANCOLOMBIA` — reescrito, reemplaza el diseño por `paymentMethod` original | `backend/CLAUDE.md` |
| 35 | Factura Electrónica Nominal también opcional para el cliente | `admin-frontend/src/cajero/CLAUDE.md` |
| 36 | Responsividad del panel admin (Sidebar/Topbar drawer + Dashboard carruseles) | `admin-frontend/CLAUDE.md` |
| 37 | `createSaleAdmin`: categoría SPECIAL/REGULAR por tope diario, gateada por `dianResponsible` | `backend/CLAUDE.md` |
| 38 | Gastos: editar/eliminar es borrado real, sin reversar nada (a diferencia de Sale/Purchase) | `backend/CLAUDE.md` |
| 39 | Zona de cajero bloqueada desde el celular (`MobileBlockScreen`) | `admin-frontend/src/cajero/CLAUDE.md` |
| 40 | "Olvidé mi contraseña" (ADMIN/MANAGER): token SHA-256 de un solo uso + correo por Resend (Render bloquea SMTP saliente) | `backend/CLAUDE.md` y `admin-frontend/CLAUDE.md` |
| 41 | Fotos de producto: Cloudinary en vez de disco local + sharp | `backend/CLAUDE.md` (detalle) y `admin-frontend/CLAUDE.md` punto 11 |
| 42 | `/login` code-split con `React.lazy` — el bundle de entrada bajó de 1MB a 378KB | `admin-frontend/CLAUDE.md` |
| 43 | `<AuthProvider>` como única fuente de la sesión — `/login` ya no espera `GET /me`, y se eliminaron hasta 4 llamadas duplicadas | `admin-frontend/CLAUDE.md` (y punto 12 en `admin-frontend/src/cajero/CLAUDE.md`) |
| 44 | `ShiftRequiredNotice` con tarjeta oscura + logo (fix de contraste), "Finalizar turno" como CTA visible en el header en vez de un ícono en `PaymentPanel` | `admin-frontend/src/cajero/CLAUDE.md` |
| 45 | Merma de stock (`StockLoss`) — botón 🗑️ en `PaymentPanel`, reduce `ProductStock` sin venta detrás | `admin-frontend/src/cajero/CLAUDE.md` |
| 46 | Catálogo del grid de Caja pagina y busca (nombre/SKU) desde el backend, ya no trae todo de una vez | `admin-frontend/src/cajero/CLAUDE.md` |
| 47 | Verificación de stock por fila (`StockVerificationV2.tsx`, confirmar/corregir con ✓/✗) reemplazó al diseño original — ya no coexisten | `admin-frontend/src/cajero/CLAUDE.md` |
| 48 | `ShiftSummary` rediseñado como hoja de cálculo; Compras en efectivo se restan del efectivo esperado; "Reporte X" eliminado del cierre | `admin-frontend/src/cajero/CLAUDE.md` |
| 49 | `react-toastify` (dependencia nueva) — toasts de éxito al abrir/cerrar turno en `ShiftModal.tsx`, `<ToastContainer />` en `CashierLayout.tsx` | `admin-frontend/src/cajero/CLAUDE.md` |
| 50 | Botón ✕ de cierre en la esquina de `ShiftModal.tsx`, fuera del switch de pasos — un solo botón para los 4 pasos | `admin-frontend/src/cajero/CLAUDE.md` |
| 51 | "Ver" en Finanzas > Caja — detalle de solo lectura de un turno (`CashClosureDetailModal.tsx`, `GET /cash-closures/:id/detail`) | `admin-frontend/CLAUDE.md` |
| 52 | Onboarding interactivo por rol (`driver.js`) — `User.hasCompletedOnboarding`, `PATCH .../auth/onboarding-complete` (admin y pos) | `backend/CLAUDE.md`, `admin-frontend/CLAUDE.md` y `admin-frontend/src/cajero/CLAUDE.md` |
| 53 | Confirmación en bloque de pagos DiDi/Rappi (miércoles de liquidación) — `PATCH /sales/confirm-payment-bulk`, banner + selección en `Ventas.tsx` | `admin-frontend/CLAUDE.md` |
| 62 | Widget "Stock Crítico" (`Product.minStock`, `getCriticalStockProducts`, `lowStockOnly`) y "Gastos por categoría" de dona a barras | `admin-frontend/CLAUDE.md` |
| 63 | Filtros colapsados en celular — buscador + modal "Más filtros" (`MoreFiltersModal.tsx`), 6 páginas (no FinanzasCaja, ver corrección) | `admin-frontend/CLAUDE.md` |
| 64 | Topbar no se actualizaba al crear/editar sedes — evento global `mecatos:branches-changed` | `admin-frontend/CLAUDE.md` |
| 65 | El negocio no declara/cobra impuestos — `Sale.tax` fijo en 0, sin IVA/Impoconsumo en recibos ni en el ticket térmico | `backend/CLAUDE.md` |
| 54 | KPIs Ventas/Compras/Gastos/Rentabilidad en el Dashboard — `summary.totalPurchases`/`totalExpenses`/`profitability` en `GET /dashboard/metrics` | `admin-frontend/CLAUDE.md` |
| 55 | Compras/Gastos del turno en el detalle de un cierre de caja — `purchases`/`expenses` en `GET /cash-closures/:id/detail`, tablas nuevas en `CashClosureDetailModal.tsx` | `admin-frontend/CLAUDE.md` |
| 56 | Rediseño de Reportes (`reportController.ts`) — logo, tabla de Inventario nueva, estilo "profesional" del PDF (pdfkit) y Excel (exceljs) | `admin-frontend/CLAUDE.md` |
| 57 | Zoom automático de iOS Safari al enfocar los inputs de `Login.tsx`/`ResetPassword.tsx` — `text-sm` (14px) subido a `text-base` (16px) | `admin-frontend/CLAUDE.md` |
| 58 | `GET /health` sin consulta a Mongo — probe liviano para un monitor externo (UptimeRobot) y evitar cold starts de Render | `backend/CLAUDE.md` |
| 59 | Correo de bienvenida al crear ADMIN/MANAGER — reutiliza el link seguro de "olvidé mi contraseña" (`generateResetToken()`), nunca manda la contraseña en texto plano | `backend/CLAUDE.md` |
| 60 | `ManagedStockModal.tsx` — ADMIN fija el stock exacto por sede (`$set`, `PUT /products/:id/stock`), distinto del top-up aditivo de `StockModal.tsx` (`$inc`) | `admin-frontend/CLAUDE.md` |
| 61 | Método de pago CARD (Tarjeta/Datáfono) quitado de toda la UI — el negocio solo recibe Efectivo/Nequi/Delivery Apps; el enum del backend lo sigue aceptando por compatibilidad con ventas históricas | `admin-frontend/CLAUDE.md` |
| 66 | Carga Masiva de inventario — matriz producto x sede (`BulkStockModal.tsx`), `$set` exacto sobre muchos productos a la vez vía `PUT /products/stock/bulk`, extensión multi-producto del punto 60 | `backend/CLAUDE.md` |
| 67 | `dianWorker.ts` lleva un servidor HTTP mínimo (`GET /health`) para desplegarse como Web Service gratis de Render en vez de un Background Worker de pago — riesgo de suspensión por inactividad aceptado explícitamente | `backend/CLAUDE.md` |
| 68 | Auditoría de seguridad — rate limiting en logins/recuperación, `trust proxy`, rechazo de operadores NoSQL, chequeo de `Origin`, validación de arranque, print-server aislado a 127.0.0.1 con CORS restringido | *(este archivo)*, `backend/CLAUDE.md`, `print-server/CLAUDE.md` y `admin-frontend/CLAUDE.md` |

## Stack

- **Backend:** Node + Express + TypeScript + Mongoose (MongoDB) + BullMQ (Redis)
  + Multer/Sharp (subida y optimización de imágenes) + cookie-parser
  (lectura de cookies httpOnly de sesión, ver punto 13). Detalle en
  `backend/CLAUDE.md`.
- **admin-frontend (única app):** React + TypeScript + Vite + TailwindCSS +
  React Router v6 (rutas admin) + Zustand (estado de la orden del cajero) y
  Dexie (solo dentro de `src/cajero/` — caché del catálogo cuando no hay
  red, y drenaje de tickets de venta encolados de ANTES del cambio
  documentado en el punto 8; ya no se crean ventas nuevas offline) +
  Recharts + **axios** (dos instancias con `withCredentials:true`, ver
  punto 13 — NO se usa `fetch` en ningún cliente API de este proyecto).
  Detalle en `admin-frontend/CLAUDE.md` y `admin-frontend/src/cajero/CLAUDE.md`.
- **pos-frontend (legado):** mismo stack que la zona cajero, sin router,
  desincronizado del código activo — ver nota arriba.

## Comandos esenciales

```bash
# Backend (requiere Mongo + Redis corriendo — ver docker-compose.yml en la raíz)
cd backend && npm install
npm run seed            # datos de prueba: sede, admin, cajero, catálogo
npm run dev              # API en :4000
npm run worker:dian      # OBLIGATORIO en proceso aparte — sin esto las ventas
                           # quedan dianStatus=PENDING para siempre
npm run reconcile:dian   # corrida manual del job de reconciliación
npm run worker:sheets    # opcional, en proceso aparte — sin esto no se
                           # sincroniza nada a Google Sheets (ver punto 21),
                           # pero el resto del sistema funciona igual

# App unificada (admin + cajero, un solo comando)
cd admin-frontend && npm install && npm run dev   # :5174
# entra a /login → elige "Soy Cajero" o "Soy Administrador"

# Todo junto (self-hosted)
docker compose up -d --build
```

Credenciales de prueba tras `npm run seed`: admin `admin@mecatoselsanti.com` /
`admin1234`; gerente de sede `gerente@mecatoselsanti.com` / `gerente1234`
(atado a la sede sembrada); cajero PIN `1234` en la sede sembrada (elige esa
sede en el selector de login de cajero).

## Decisiones universales (aplican a todo el proyecto)

### 6. Auth dual en el backend, cookies httpOnly en el frontend (no headers manuales)

Backend:
- **Admin/Manager:** JWT verificado por `middlewares/adminAuth.ts`
  (`requireAdminAuth` + `requireRole(...)`). Rutas bajo `/api/admin/*`.
- **Cajero:** sesión ligera por PIN de 4 dígitos + sede, verificada por
  `middlewares/posAuth.ts` (`requirePosSession`). Rutas bajo `/api/pos/*`.
  El payload incluye `loginAt` (ISO date) — úsalo para filtrar "de esta
  sesión", no `createdAt` de hoy a secas (ver punto 12).

**Desde la migración a cookies (ver punto 13), ambos middlewares leen el
JWT de `req.cookies.admin_token` / `req.cookies.cashier_token` — ya NO leen
el header `Authorization`.** Si escribes un test o un script que le pegue a
la API directamente, tendrás que simular la cookie, no un Bearer token.

No uses `requireAdminAuth` en rutas de POS ni viceversa — son modelos de
sesión distintos con payloads distintos (`req.admin` vs `req.posSession`),
y cookies con nombres distintos.

En el frontend (`admin-frontend`, una sola app), cada zona tiene su propio
cliente axios (`src/services/httpClient.ts` para admin,
`src/cajero/services/httpClient.ts` para cajero), ambos con
`withCredentials: true`. No mezcles: un componente de `src/cajero/` nunca
debe importar `src/services/api.ts`, y viceversa.

### 13. Autenticación: axios + cookies httpOnly, NO localStorage, NO fetch manual

Esto reemplazó un diseño anterior (fetch + Bearer token en localStorage).
Si ves código o instrucciones viejas que mencionen `localStorage.getItem/
setItem("admin_token"/"cashier_token"/"cashier_session")`, están obsoletas
— no las repliques.

**Cómo funciona ahora:**
- El backend, en login (`adminLogin`/`posLogin` en `authController.ts`),
  firma el JWT igual que antes pero lo pone en una cookie httpOnly con
  `setAuthCookie()` (`utils/cookies.ts`) en vez de devolverlo en el JSON de
  respuesta. El body de la respuesta de login solo trae datos de perfil, no
  el token.
- `middlewares/adminAuth.ts` y `middlewares/posAuth.ts` leen el JWT de
  `req.cookies.admin_token` / `req.cookies.cashier_token` (vía
  `cookie-parser`, montado en `app.ts`), no del header `Authorization`.
- `POST /api/admin/auth/logout` y `POST /api/pos/auth/logout` limpian la
  cookie del lado del servidor (`clearAuthCookie()`) — el frontend no puede
  hacerlo directamente porque `httpOnly` bloquea el acceso desde JS
  incluso para borrarla.
- `GET /api/admin/auth/me` y `GET /api/pos/auth/me` son la única forma que
  tiene el frontend de saber "¿hay sesión activa?", porque no puede leer la
  cookie. `RequireAuth`, `RequireCashierAuth`, `RootRedirect` y
  `CashierLayout` llaman a estos endpoints (con un estado `loading`
  intermedio) en vez de hacer un chequeo síncrono de localStorage.
- **CORS con `credentials: true` es obligatorio** para que el navegador
  mande/reciba las cookies en peticiones cross-origin — `origin` en
  `app.ts` NO puede ser `"*"`, viene de `CORS_ORIGIN` (env var, lista
  separada por comas). Si agregas un nuevo frontend/origen que consuma esta
  API, agrégalo a `CORS_ORIGIN`, no cambies a `origin: "*"`.
- **`ms` package** calcula el `maxAge` de la cookie a partir del mismo
  string que usa `expiresIn` del JWT (`JWT_EXPIRES_IN`/
  `POS_SESSION_EXPIRES_IN`) — así ambos quedan sincronizados
  automáticamente. Si cambias la duración de una sesión, solo toca esa
  variable de entorno; no hay un número mágico duplicado en otro archivo.

**En el frontend, cada zona tiene su propio cliente axios** con
`withCredentials: true` (`services/httpClient.ts` para admin,
`cajero/services/httpClient.ts` para cajero) — instánciales una sola vez
ahí, no crees `axios.create()` sueltos en componentes. Ambos tienen un
interceptor de respuesta que normaliza errores a `new Error(mensaje)` (así
`catch((err) => err.message)` sigue funcionando igual que antes) y
redirige a `/login` en un 401 que no sea de `/auth/login` ni `/auth/me`
(para no generar loops de redirect durante el propio chequeo de sesión).

**Gotcha de dev local:** si cambias los puertos de dev (`vite.config.ts`)
o el dominio de despliegue, tienes que actualizar `CORS_ORIGIN` en el
backend — si no coincide exactamente con el origen del frontend, el
navegador bloquea la petición (ni siquiera llega a los logs del backend
como error 401, es un bloqueo de CORS en el propio navegador, más difícil
de diagnosticar).

**Gotcha de `NODE_ENV=production` vs. `COOKIE_SECURE`:** `utils/cookies.ts`
en algún momento tuvo un bug real donde `NODE_ENV=production` forzaba
`secure:true` en la cookie **incluso si `COOKIE_SECURE=false` estaba puesta
explícitamente** — porque la lógica era `env === "true" || isProd` (el
`|| isProd` ganaba siempre). Esto rompía el login en cualquier despliegue
con `NODE_ENV=production` pero sin HTTPS todavía configurado (ej. un
`docker-compose up` recién hecho, antes de poner el reverse proxy con TLS),
porque el navegador descarta cookies `Secure` en conexiones sin TLS. Ya está
corregido: ahora una `COOKIE_SECURE` explícita (aunque sea `"false"`) tiene
prioridad sobre la inferencia por `NODE_ENV`. Si tocas esta lógica de nuevo,
no repitas el patrón `envVar === "true" || condiciónDerivada` para flags
booleanos de seguridad — siempre debe poder forzarse explícitamente en
ambas direcciones.

### 33. Todo el proyecto maneja fechas/horas en zona horaria de Colombia (`America/Bogota`, UTC-5 fijo, sin horario de verano) — nunca en la del navegador/servidor

El negocio es 100% en Colombia — ninguna fecha/hora que se MUESTRE o se
FILTRE debería depender de en qué zona horaria esté configurado el
navegador de quien lo esté viendo, ni la del SO/contenedor donde corre el
backend. Antes de esto, ninguno de los dos lados fijaba una zona horaria
explícita en la mayoría de los sitios — cada uno "funcionaba" solo porque
la máquina de turno resultaba tener el reloj correcto, no porque el
código lo garantizara. Bogotá no observa horario de verano, así que un
offset fijo (`-05:00`) es tan correcto como una librería de timezones,
sin la dependencia añadida.

**Backend — `backend/src/utils/dateRange.ts` es el único punto de
verdad** (ver punto 24 en `backend/CLAUDE.md` para el detalle de la
reescritura): exporta `COLOMBIA_TIME_ZONE` ("America/Bogota"),
`COLOMBIA_UTC_OFFSET` ("-05:00"), `startOfLocalDay`/`endOfLocalDay` (para
un string "YYYY-MM-DD" de un `<input type="date">`) y
`getStartOfTodayColombia()`/`getTodayColombiaDateString()` (para "ahora
mismo, ¿qué día es hoy en Bogotá?"). Se reemplazaron todos los `new
Date(); d.setHours(0,0,0,0)` que calculaban "inicio de hoy" a mano —
`getDashboardKpis`/`getDashboardMetrics` (`adminController.ts`),
`getDailyTotal` (`posController.ts`, el control de tope diario de venta
del cajero), y el fallback de `getCashierShiftStart`
(`utils/shiftRange.ts`) — por `getStartOfTodayColombia()`. Si agregas un
cálculo nuevo de "hoy"/rango de fechas en el backend, usa estas
funciones — no repitas `new Date().setHours(...)` a mano, eso depende
silenciosamente de la zona horaria del proceso de Node (que ningún
Dockerfile de este proyecto fija — `node:20-alpine`, la imagen base, cae
a UTC por defecto).

**Defensa adicional (no la corrección en sí):** `docker-compose.yml` fija
`TZ: America/Bogota` en los tres servicios de Node (`backend`,
`dian-worker`, `sheets-worker`) — esto no es necesario para que la lógica
de arriba funcione (todo usa offsets/zona explícitos, no depende de
`process.env.TZ`), pero es una red de seguridad barata para cualquier
`toLocaleString()`/`Date` que alguien escriba a futuro sin poner
`timeZone` explícito.

**Backend — formateo de fechas para EXPORTAR (Excel/PDF), no para
filtrar:** `reportController.ts` tenía 6 sitios con
`new Date(x).toLocaleString("es-CO")` sin `timeZone` (columnas "Fecha" del
Excel/PDF del reporte financiero, más el sello "Generado: ...") — ahora
todos pasan por un helper compartido `formatBogota()` dentro del mismo
archivo, que sí fija `timeZone: COLOMBIA_TIME_ZONE`.

**Frontend (`admin-frontend`) — `src/utils/timezone.ts` es el único punto
de verdad**, con una copia intencional en `src/cajero/utils/timezone.ts`
(mismo patrón de duplicación que `printerService.ts`/`SaleReceipt.tsx`,
ver punto 12 en `admin-frontend/src/cajero/CLAUDE.md` — la copia de cajero
omite los helpers de aritmética de rango de fechas porque esa zona no
tiene selectores de rango, solo la necesita el panel admin). Expone:
- `formatDateTime`/`formatDate`/`formatTime` — reemplazo directo de
  `new Date(x).toLocaleString/toLocaleDateString/toLocaleTimeString("es-CO")`
  sin `timeZone`, que antes dependía silenciosamente del reloj/zona
  horaria del navegador de quien estuviera viendo la pantalla (un admin
  revisando el panel desde otro país vería una hora de venta distinta a
  la que el cajero vio al imprimir el ticket).
- `todayColombia()` — reemplazo de
  `new Date().toISOString().slice(0, 10)`, el bug que motivó todo este
  punto: ese patrón da el día calendario en **UTC**, no en Bogotá, así que
  después de las 7pm hora Bogotá (UTC-5, cuando UTC ya cruzó a las 00:00
  del día siguiente) "Hoy" mostraba silenciosamente la fecha de mañana.
  Encontrado primero en `Dashboard.tsx` (el selector "Hoy/Ayer/Esta
  semana/Este mes"), y con el mismo patrón exacto en
  `FinanzasReportes.tsx` (los date pickers "Desde"/"Hasta" por defecto).
- `addDaysToDateString`/`dayOfWeekForDateString`/`firstOfMonthForDateString`
  (solo en la copia del panel admin) — aritmética de calendario sobre
  strings "YYYY-MM-DD" vía `Date.UTC`, para que "Ayer"/"Esta semana"/"Este
  mes" en `Dashboard.tsx` no vuelvan a depender de `Date` + getters/
  setters locales del navegador.

Call sites migrados a estos helpers: `Dashboard.tsx` (`computeRange`
completo), `FinanzasReportes.tsx` (`todayInput`/`firstOfMonthInput`),
`SaleReceipt.tsx` (ambas copias, fecha/hora del recibo),
`cajero/pages/Facturas.tsx` (hora en la lista de facturas), `Ventas.tsx`
y `Compras.tsx` (columna "Fecha"), `CuentasPorCobrar.tsx` y
`CuentasPorPagar.tsx` (columna "Vence" — especialmente sensible si
`dueDate` se guarda como medianoche UTC de un día calendario, ya que sin
`timeZone` explícito un navegador en un huso horario distinto podía
mostrar el día anterior). `pos-frontend/` (legado, ver su propio README)
tiene el mismo patrón sin corregir — no se tocó a propósito, ver la nota
de alcance en "Pendiente conocido".

**`print-server/index.js`** (ver punto 32 en `print-server/CLAUDE.md`)
también tenía dos `new Date().toLocaleString("es-CO")` sin `timeZone` (la
fecha impresa en el ticket térmico) — ahora pasan por un
`formatBogotaDateTime()` local con `timeZone: "America/Bogota"` explícito.
Importa acá en particular porque este servicio corre en la máquina física
de la tienda, cuyo reloj/zona horaria del sistema operativo es la menos
controlada de todo el stack.

**Si agregas cualquier fecha nueva que se muestre o se filtre en este
proyecto**, pasa por estos helpers (`backend/src/utils/dateRange.ts` o
`admin-frontend/src/utils/timezone.ts`/`cajero/utils/timezone.ts`) — no
repitas `new Date().toLocaleString(...)` sin `timeZone`, ni
`toISOString().slice(0, 10)`, ni `new Date(); .setHours(...)` a mano.

### 68. Auditoría de seguridad (backend + print-server + headers del frontend) — qué existe, qué flags lo controlan, y qué NO se cerró

Resultado de una auditoría de los tres paquetes. El detalle de implementación
vive en `backend/CLAUDE.md` (punto 68), `print-server/CLAUDE.md` (punto 68) y
`admin-frontend/CLAUDE.md` (punto 68); acá solo lo que aplica a todo el
proyecto y **las variables de entorno nuevas**. **Rutas reales** (el pedido de
la auditoría hablaba de `/api/auth/login`, `/api/auth/pin-login`,
`/api/auth/forgot-password` — esas rutas no existen): admin/gerente entra por
`POST /api/admin/auth/login`, recupera con `POST /api/admin/auth/forgot-password`
y `.../reset-password`, y el cajero (sede + PIN) por `POST /api/pos/auth/login`.

Lo que se implementó:
- **Rate limiting** (`express-rate-limit`, memoria) en login admin (10 fallidos/15 min
  por IP+correo, más 20 por correo desde cualquier IP — ver el incidente de Render en `backend/CLAUDE.md` 68), login por PIN (10 fallidos/15 min por IP **y** 30 por sede — un PIN
  de 4 dígitos son solo 10.000 combinaciones), forgot-password (5/h por IP y 3/h
  por correo) y reset-password (10/15 min por IP). Responde 429.
- **`trust proxy`** configurable (`TRUST_PROXY`, por defecto `1` en producción) — sin
  esto, detrás de Render `req.ip` es el del balanceador y el limitador no distingue clientes.
- **Anti-inyección NoSQL**: middleware global que responde 400 si body/query/params
  traen claves `$…` o con punto, más validación con `zod` (tipos `string`) en los
  endpoints de auth.
- **Anti-CSRF por `Origin`** en peticiones que modifican datos (necesario porque
  `SameSite=None` manda las cookies también desde sitios ajenos).
- **Validación al arrancar** (`config/security.ts`): sin `JWT_SECRET`/`POS_SESSION_SECRET`,
  con `CORS_ORIGIN="*"` o sin `CORS_ORIGIN` en producción, el backend **no arranca**.
  Secretos cortos/de ejemplo o `COOKIE_SECURE=false` en producción solo **advierten** —
  `SECURITY_STRICT=true` los vuelve errores.
- **print-server**: solo `127.0.0.1`, CORS con lista de orígenes, chequeo de `Host`,
  validación de la forma del payload, límite de tamaño.
- **Otros**: JWT verificado con `algorithms: ["HS256"]`, PIN de cajero exigido de 4
  dígitos también al crear/editar, comparación bcrypt "falsa" en login para no
  filtrar por tiempo si un correo existe, errores 5xx sin `err.message` en producción,
  `Cache-Control: no-store` en `/api`, Mongo/Redis del `docker-compose` publicados solo
  en `127.0.0.1`, headers de seguridad en `admin-frontend/vercel.json`, y
  `npm audit fix` (sin `--force`) en el backend.

**Variables de entorno nuevas / relevantes:**

| Variable | Dónde | Efecto |
|---|---|---|
| `TRUST_PROXY` | backend | Saltos de proxy (`1` en Render). Por defecto `1` si `NODE_ENV=production`, `false` si no. |
| `SECURITY_STRICT` | backend | `true` = las advertencias de configuración (secreto corto, cookie sin `Secure`) detienen el arranque. |
| `CORS_ORIGIN` | backend | Ya existía. Ahora también valida el `Origin` de POST/PUT/PATCH/DELETE y es **obligatoria** en producción. |
| `PRINT_ALLOWED_ORIGINS` | print-server (`print-server/.env`) | Orígenes del frontend que pueden imprimir (por defecto solo `localhost:5174`). **En producción hay que agregar el dominio desplegado en ese `.env`**, o la impresión directa se bloquea y el frontend cae al diálogo del navegador. |

**Lo que NO se cerró (a propósito o por alcance) — no lo asumas resuelto:**
- **Los JWT no se revalidan contra la base en cada request**: un usuario desactivado
  conserva su sesión hasta que expire el token (8 h admin, 12 h cajero); solo `GET /me`
  comprueba `active`. Cerrarlo cuesta una consulta por request.
- **Asignación masiva**: `updateProduct`/`updateBranch` hacen `findByIdAndUpdate(id, req.body)`
  — Mongoose descarta campos que no están en el esquema, pero un gerente puede editar
  cualquier campo del producto (p. ej. `siigoCode`, `minStock`). Una lista blanca de campos sigue pendiente.
- **Sin CSP en el frontend** (solo los headers básicos en `vercel.json`): una CSP
  incorrecta rompería el sitio en producción (dominio de la API, fuentes, Cloudinary,
  `localhost:4001`) y no se pudo probar contra el despliegue real.
- **El rate limiter usa memoria**: correcto con una sola instancia (Render gratis);
  con varias habría que moverlo a Redis (costaría comandos de Upstash, ver punto 4).
- **`npm audit`**: queda 1 moderada en el backend (`uuid` vía `exceljs`, exige bajar
  `exceljs` a una versión mayor anterior). `print-server/` **tiene `node_modules/`
  versionado en su repo** (sin `.gitignore`) — conviene sacarlo del control de versiones.
- **Secretos de desarrollo**: el `.env` local tiene un `POS_SESSION_SECRET` corto, y el
  `docker-compose.yml` cae a `cambia_este_secreto*` si no defines los tuyos — nunca lo
  uses así en un despliegue real.

## Convenciones de código

- Español para nombres de dominio de negocio en UI/comentarios (Sedes,
  Cajero, Arqueo), inglés para nombres técnicos genéricos (Service, Queue,
  Handler). Sigue lo que ya existe en el archivo que estés editando.
- Comentarios explicando el *porqué* de una decisión no obvia (ver ejemplos
  en los `CLAUDE.md` de cada paquete), no explicando *qué* hace una línea
  de código evidente.
- Tailwind con clases utilitarias inline, sin CSS-in-JS ni styled-components.
- Formularios de admin: modal dedicado por entidad, no formulario inline
  (ver punto 17 en `admin-frontend/CLAUDE.md`) — botón "+ Nuevo X" abre el
  modal, `DataTable` compartido (o tabla propia) para listar. Sigue ese
  patrón al agregar páginas nuevas en vez de inventar uno distinto.

## Pendiente conocido (no asumas que ya existe)

- **Integración real con Siigo (PTA) — funcionando y ya probada contra la
  cuenta de producción, `DIAN_PROVIDER=SIIGO` activo en este ambiente de
  dev**: ver punto 10 en `backend/CLAUDE.md` para el detalle completo.
  `dianService.ts` habla con la cuenta real de Siigo (`Partner-Id:
  MecatosElSanti`), el mapeo por sede (`Branch.dianConfig.siigoSellerId`/
  `siigoDocumentId`) y el catálogo de 44 productos reales (`Product.siigoCode`)
  ya están cargados, y se confirmó contra facturas reales tanto la forma de
  la respuesta (`cufe` en `stamp.cufe`, el link de factura/QR en
  `public_url` — no `stamp.qr_code`) como que el timbrado es asíncrono
  (`siigoEmit` reconsulta la factura hasta confirmar el CUFE antes de
  darse por vencido, ver punto 10).
  **⚠️ Ya se generaron facturas reales, timbradas ante la DIAN, durante
  las pruebas** — algunas sin venta real detrás; anularlas requiere una
  Nota Crédito en Siigo por cada una (una `Sale` cancelada/borrada en
  Mongo no anula nada del lado de Siigo/DIAN). Antes de volver a probar
  con `DIAN_PROVIDER=SIIGO`, considera si de verdad quieres generar otra
  factura real — no hay ambiente sandbox separado con estas credenciales.
  **Resuelto**: el mapeo de pago para ventas de delivery apps ya no
  depende de `SIIGO_PAYMENT_ID_DELIVERY_APP` — el refactor del punto 34
  (`Sale.paymentMethod` ahora incluye `EFECTIVO`/`BANCOLOMBIA`, ver ese
  punto) mapea `BANCOLOMBIA` a `SIIGO_PAYMENT_ID_BANCOLOMBIA=105`
  ("Transferencia Bancolombia", ya existía en el catálogo real, sin usar
  hasta ahora). **Todavía pendiente**: mapeo de `SIIGO_PAYMENT_ID_CARD`
  (sin id claro en el catálogo real de la cuenta — decisión de negocio, no
  un olvido, e irrelevante en la práctica ya que `CARD` es legacy-only) y
  un campo en `ProductModal.tsx` para editar `siigoCode` desde la UI (hoy
  se edita directo en Mongo).
  **⚠️ Deuda técnica nueva, a propósito**: `createBranch` defaultea
  `dianConfig.siigoSellerId`/`siigoDocumentId` de toda sede nueva
  `dianResponsible` al vendedor/documento de "Boulevard" (`1032`/`32502`)
  en vez de exigir uno real por sede — sin esto, una sede nueva quedaba
  sin poder emitir ninguna venta hasta configurarla a mano en
  `DianConfig.tsx`. Efecto real: mientras el negocio solo opere una sede
  `dianResponsible` de verdad esto no se nota, pero en cuanto abra una
  segunda, ambas quedarían facturando ante la DIAN bajo el mismo
  vendedor/resolución de Siigo hasta que alguien dé de alta uno real para
  la sede nueva y lo actualice a mano en `DianConfig.tsx` (ver punto 10 en
  `backend/CLAUDE.md` para el detalle completo).
- Webhooks reales de Rappi/DiDi (Hubster/Deliverect) — no implementados.
- KDS (Kitchen Display System) vía WebSockets — no implementado. La
  impresión térmica de recibos SÍ está implementada (ver punto 32 en
  `print-server/CLAUDE.md`) — no la confundas con esto.
- Kardex con descuento automático de insumos por receta (BOM) al vender — el
  modelo `recipe` existe en `Product` pero no se descuenta stock aún.
- ~~Migración de fotos a object storage~~ — **hecho para fotos de
  producto** (Cloudinary, ver punto 41). Las 4 fotos subidas antes de esa
  migración se quedaron en disco local a propósito (no se re-subieron,
  ver el detalle en el punto 41) — si se quiere una migración 100%
  completa de esas, sigue pendiente. No existe hoy ninguna subida real de
  "foto de recibo" en el código (ver la corrección en el punto 41 sobre
  ese comentario viejo, que era aspiracional) — si se agrega esa feature
  a futuro, debería usar Cloudinary desde el principio, no disco local.
- Tests automatizados — no hay ninguno todavía.
- `pos-frontend/` sigue en el repo como legado desincronizado (ver su
  README) — nadie lo mantiene activamente. No asumas que refleja el
  comportamiento actual del sistema; la fuente de verdad es
  `admin-frontend/src/cajero/`.
- El módulo "Compras" del cajero (`Purchase` model) ya NO es informal — ver
  punto 16 en `admin-frontend/CLAUDE.md` (reemplazado por un flujo
  producto/cantidad/monto ligado a inventario, igual que el del admin).
  Ninguno de los dos flujos toca `AccountPayable` — si el negocio pide que
  una compra grande se convierta automáticamente en cuenta por pagar, eso
  sigue siendo una feature nueva, no algo que ya exista.
- `pos-frontend/` (legado) **no fue migrado** a axios/cookies httpOnly —
  sigue con `fetch` + token en localStorage, tal como quedó antes de la
  migración de auth. No lo copies como referencia para el patrón de auth
  actual; la fuente de verdad es `admin-frontend`. Tampoco se migró al
  consolidar el manejo de zona horaria de Colombia (ver punto 33 arriba) —
  `InvoicesModal.tsx` sigue formateando fechas sin `timeZone` explícito.
  No se tocó a propósito (nadie mantiene este paquete activamente); si en
  algún momento se pide corregirlo, replica `admin-frontend/src/cajero/
  utils/timezone.ts` ahí.
- La configuración actual de cookies (`COOKIE_SAMESITE`/`COOKIE_SECURE`) se
  probó en dev con front y back en el mismo dominio (`localhost`, distinto
  puerto). Si el despliegue real termina con frontend y backend en dominios
  completamente distintos, verifica en un entorno real que
  `SameSite=None; Secure` funcione como se espera antes de dar por hecho
  que "ya está resuelto" — cookies cross-site tienen más matices por
  navegador (Safari en particular ha sido históricamente más estricto) de
  los que este proyecto ha podido probar.
- La sincronización con Google Sheets (ver punto 21 en `backend/CLAUDE.md`
  y `docs/GOOGLE_SHEETS_INTEGRATION.md`) no tiene job de reconciliación
  como DIAN — un job que agote sus 5 reintentos queda `failed` sin
  reintento automático posterior. Tampoco se ha probado bajo carga real
  sostenida (muchas ventas/compras por minuto); lo único verificado es que
  Apps Script puede tardar/serializar bajo ráfagas cortas de pocos
  requests seguidos. Si el volumen real de la panadería termina siendo
  alto, esto puede necesitar un job de reconciliación propio o pasar a
  otro medio (no necesariamente Apps Script) más adelante.
