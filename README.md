# Mecatos el Santi — ERP/POS Multisede

Sistema integral: backend (API + colas) y un frontend único
(`admin-frontend`) con **login diferenciado por rol** — cajeros y
administradores entran por la misma pantalla de login y cada uno llega a su
propia zona de la app.

## Estructura del monorepo

```
mecatos-erp/
├── backend/            API REST + colas BullMQ + worker de emisión DIAN
├── admin-frontend/      App unificada: login por rol, panel admin y zona de cajero
├── pos-frontend/        ⚠️ Legado/opcional — ver pos-frontend/README.md
└── print-server/        Impresión térmica local (opcional) — ver print-server/README.md
```

### Login diferenciado por rol

`admin-frontend/src/pages/Login.tsx` es el único punto de entrada. Presenta
dos opciones — "Soy Cajero" (PIN + selección de sede) y "Soy Administrador"
(correo + contraseña) — y cada una navega a una zona distinta de la misma
app, con guardas de ruta (`RequireAuth` para admin, `RequireCashierAuth`
para cajero) y sesiones separadas vía cookies httpOnly (`admin_token` vs.
`cashier_token` — ver la sección de cookies más abajo, ninguna de las dos
toca `localStorage`). El chequeo de sesión inicial (`GET /auth/me`) corre
una sola vez al cargar la app, en un `<AuthProvider>` compartido — las
guardas de ruta solo leen ese resultado, no vuelven a pedirlo cada una por
su cuenta (ver punto 43 en `admin-frontend/CLAUDE.md`). `/login` en sí
nunca espera a ese chequeo para mostrarse: si termina descubriendo que ya
había una sesión activa, redirige lejos en cuanto lo confirma.
También se puede mostrar/ocultar la contraseña con el ícono de ojo en el
formulario de administrador.

### Zona del cajero: 3 pestañas

Al iniciar sesión con PIN, el cajero llega a `/cajero/*` con una barra de
pestañas fija (`CashierLayout.tsx`), que también tiene un botón "Finalizar
turno" centrado en el header (mismo estilo que "Iniciar turno" del aviso
de abajo) para cerrar el turno desde cualquier pestaña:

- **Caja** (`/cajero/caja`) — el POS de siempre: grid de productos (solo
  los que tienen stock disponible en la sede), paginado y con un buscador
  por nombre/código junto a las pestañas de categoría (ambos manejados
  desde el backend, no en el cliente), orden activa, pagos de un clic,
  modals de gasto menor, merma de stock (producto dañado/vencido,
  consumo interno) y arqueo. Si el cajero no tiene un turno
  abierto, Caja/Facturas/Compras muestran un aviso con un botón "Iniciar
  turno" en vez de sus datos reales — pero navegar entre pestañas y
  cerrar sesión siguen funcionando siempre, con o sin turno abierto. El
  arqueo (apertura y cierre de
  turno) ahora también incluye una verificación de stock: una tabla con el
  stock actual de la sede (SKU, precio, cantidad, valor total) que el
  cajero confirma o, si no coincide, anota la diferencia — además del
  conteo de efectivo y Nequi de siempre. Es el mismo código que antes vivía
  en `pos-frontend`, ahora en `admin-frontend/src/cajero/`. Toda venta se
  crea en línea y valida/descuenta stock al confirmarse — si falla (sin
  conexión, stock insuficiente), se muestra el error y la orden queda
  intacta para reintentar, en vez de encolarse offline (IndexedDB/Dexie
  sigue usándose para cachear el catálogo sin red y para drenar tickets
  que hayan quedado pendientes de antes de este cambio, no para ventas
  nuevas — ver `CLAUDE.md` punto 8 para el detalle). Al confirmarse una
  venta, el recibo imprimible aparece automáticamente (mismo componente
  que "Ver recibo" en Facturas).
- **Facturas** (`/cajero/facturas`) — ventas creadas por ese cajero durante
  su **turno actual** (desde que abrió su último arqueo, sin importar si
  ya lo cerró — no por día calendario ni por login, para que un turno
  nocturno que cruza medianoche no se corte a la mitad, ni se "pierda" al
  cerrar sesión y volver a entrar), con su estado de emisión DIAN (CUFE una
  vez aprobada). Cada venta tiene un botón
  "Ver recibo" que abre el mismo recibo imprimible que usa el panel admin
  (logo, datos de la sede, ítems, subtotal/total — sin impuestos, el
  negocio no los declara ni los cobra —, `window.print()`).
- **Compras** (`/cajero/compras`) — el cajero registra compras de
  reabastecimiento (producto + cantidad + monto), restringidas a su propia
  sede, que agregan stock a `ProductStock` igual que el flujo equivalente
  del panel admin — ya no es el flujo informal de antes (proveedor/
  concepto/monto + foto de recibo, sin tocar inventario). Estas quedan
  visibles para el admin en `/compras` del panel.

### Asignación de sede al crear un cajero

Al crear un usuario en `/personal` (panel admin), el formulario incluye un
selector de **sede explícito** — independiente del filtro de la barra
superior — que determina a qué sede tendrá acceso ese cajero al iniciar
sesión con su PIN.

### Comunicación frontend↔backend: axios + cookies httpOnly

`admin-frontend` usa **axios** (no `fetch`) para todas las llamadas al
backend, y la autenticación viaja en **cookies httpOnly** — no en
`localStorage` ni en un header `Authorization` manejado por el frontend.

- El backend, al hacer login (admin o cajero), responde con
  `Set-Cookie: admin_token=...; HttpOnly` (o `cashier_token`). El navegador
  la guarda solo; ningún script del frontend puede leerla (esto es lo que
  protege el token de robo vía XSS, a diferencia de guardarlo en
  localStorage).
- Cada cliente axios (`admin-frontend/src/services/httpClient.ts` para
  admin, `admin-frontend/src/cajero/services/httpClient.ts` para cajero) usa
  `withCredentials: true`, así el navegador adjunta la cookie correspondiente
  en cada petición automáticamente.
- Como el frontend no puede leer la cookie, las guardas de ruta
  (`RequireAuth`, `RequireCashierAuth`) verifican la sesión llamando a
  `GET /auth/me` en vez de revisar `localStorage` — por eso ahora muestran un
  estado breve de "Verificando sesión..." al cargar.
- Cerrar sesión llama a `POST /auth/logout`, que es quien limpia la cookie
  (`res.clearCookie`) — el frontend no puede borrarla directamente por ser
  httpOnly.

**Requisito de CORS:** con cookies + `credentials: true`, el backend no
puede usar `origin: "*"` — debe declarar explícitamente qué origen(es)
puede llamarlo, vía la variable `CORS_ORIGIN` (ver `.env.example`).

**Nota sobre despliegue con dominios distintos:** si en producción el
frontend y el backend viven en dominios diferentes (ej.
`app.tudominio.com` y `api.tudominio.com`), la cookie necesita
`SameSite=None; Secure` — configúralo con `COOKIE_SAMESITE=none` y
`COOKIE_SECURE=true` (esto último requiere que el backend esté detrás de
HTTPS, que la mayoría de plataformas de hosting proveen por defecto).

**El `docker-compose.yml` de este repo pone `COOKIE_SECURE=false` y
`COOKIE_SAMESITE=lax` por defecto a propósito** (aunque `NODE_ENV=production`
esté fijo ahí), porque un `docker compose up` recién hecho normalmente no
tiene HTTPS todavía — si defaulteara a `secure:true`, el login se rompería
hasta que pusieras un reverse proxy con TLS delante. En cuanto tengas HTTPS
real (Nginx/Caddy/Traefik u otro), sobreescribe esas dos variables en tu
`.env` según corresponda a tu topología de dominios.

## Requisitos previos

- Node.js 20+
- MongoDB (local o Atlas)
- Redis (local o servicio administrado) — usado por BullMQ para la cola de
  emisión DIAN y, si la usas, la de sincronización con Google Sheets. Si es
  un proveedor managed tipo Upstash, la `REDIS_URL` debe usar el esquema
  `rediss://` (con TLS) — con `redis://` a secas la conexión "conecta" pero
  cada comando falla en silencio (ver `CLAUDE.md` punto 4 si ves reconexiones
  en loop sin que nada se encole).

## 1. Backend

```bash
cd backend
cp .env.example .env     # edita MONGO_URI, JWT_SECRET, CORS_ORIGIN, etc.
npm install
npm run seed              # crea sede, admin, cajero demo y catálogo de prueba
npm run dev                # levanta la API en http://localhost:4000
```

En **otra terminal**, levanta el worker que procesa la emisión DIAN en segundo plano:

```bash
cd backend
npm run worker:dian
```

> Sin el worker corriendo, las ventas quedarán en estado `dianStatus: PENDING`
> indefinidamente — el worker es quien las envía a la cola de BullMQ/Redis y
> actualiza el estado a `APPROVED`/`REJECTED`.

### Job de reconciliación DIAN

El worker (`dianWorker.ts`) incluye un job de reconciliación que corre en el
mismo proceso: al arrancar y luego cada `RECONCILE_INTERVAL_MINUTES` (5 por
defecto), busca ventas con `dianStatus: PENDING` de más de
`RECONCILE_STALE_MINUTES` (2 por defecto) de antigüedad y las vuelve a
encolar — cubre el caso en que Redis falló justo al momento del cobro (el
backend igual guarda la venta y responde al cajero, ver
`posController.createSale`) o el worker se cayó mientras procesaba un job.

Usa un `jobId` determinístico por venta (`dian-emission-<saleId>`), así que
si ya existe un job vivo (esperando/activo/en backoff) para esa venta, no se
duplica.

Para forzar una corrida manual sin esperar el ciclo automático (por ejemplo,
justo después de que Redis estuvo caído un rato):

```bash
cd backend
npm run reconcile:dian
```

> Nota: este job cubre específicamente ventas que quedaron `PENDING` sin
> haber sido encoladas. Ventas que sí se procesaron pero terminaron
> `REJECTED` tras agotar sus 5 reintentos (fallo real del proveedor DIAN, no
> de infraestructura) requieren revisión manual desde `/ventas` en el panel
> admin — no se reintentan automáticamente para evitar reenviar algo que el
> proveedor ya rechazó de forma definitiva.

### Worker de sincronización con Google Sheets (opcional)

Si quieres reflejar inventario, ventas/compras/gastos y cierres de caja en
tiempo real en una hoja de Google, hay un segundo worker opcional — ver
`docs/GOOGLE_SHEETS_INTEGRATION.md` para el detalle completo (Apps Script a
desplegar, estructura de las 3 pestañas, variables de entorno). En resumen:

```bash
cd backend
npm run worker:sheets
```

Necesita `GOOGLE_SHEETS_WEBHOOK_URL` definida (la URL `/exec` del Apps
Script Web App que despliegas en tu Google Sheet). Sin esta variable, o sin
el worker corriendo, el resto del sistema funciona exactamente igual — la
sincronización a Sheets es una capa de reporting aparte, no algo de lo que
dependa ninguna operación real (ventas, stock, etc. se guardan en Mongo sin
importar si esto está corriendo).

**Credenciales de prueba (después de `npm run seed`):**
- Admin: `admin@mecatoselsanti.com` / `admin1234`
- Cajero: sede *"Mecatos el Santi — Sede Centro"*, PIN `1234`

## 2. App unificada (admin-frontend)

```bash
cd admin-frontend
cp .env.example .env      # VITE_API_URL y VITE_POS_API_URL apuntando al backend
npm install
npm run dev                 # http://localhost:5174
```

Entra a `http://localhost:5174/login` y elige "Soy Cajero" o "Soy
Administrador" según con qué credenciales de prueba quieras entrar.

## 3. Impresión térmica (opcional, solo si hay impresora física)

`print-server/` es un servicio Express aparte de `backend/` — corre en
la máquina física que tiene la impresora térmica USB conectada (la
terminal del cajero, o del admin), en el puerto **4001**:

```bash
cd print-server
npm install   # ver print-server/README.md — libusb como prerrequisito
npm start     # http://localhost:4001
```

No es necesario para que el resto del sistema funcione: el botón
"Imprimir recibo" (panel admin y cajero) intenta este servicio local
primero y cae automáticamente al diálogo de impresión del navegador
(`window.print()`) si no está corriendo — no hace falta arrancarlo en
desarrollo ni en ninguna terminal sin impresora física. Ver
`docs/THERMAL_PRINTER_INTEGRATION.md` para el diseño completo.

## Despliegue en producción

### Redis

El backend usa Redis (vía BullMQ) para la cola de emisión DIAN. En producción
**no instales Redis a mano en el servidor** — usa una de estas dos vías:

**A) Redis administrado (recomendado)** — Upstash, Redis Cloud, AWS ElastiCache,
Azure Cache for Redis, o el addon de Redis de Railway/Render. Todos te dan una
única `REDIS_URL` (normalmente con TLS, prefijo `rediss://`). Solo necesitas
definir esa variable:

```bash
REDIS_URL=rediss://default:tu_password@tu-host.upstash.io:6379
```

`backend/src/config/redis.ts` detecta automáticamente `REDIS_URL` y la usa en
lugar de `REDIS_HOST`/`REDIS_PORT`/`REDIS_PASSWORD`.

**B) Redis autogestionado con Docker** — usa el `docker-compose.yml` en la raíz
del monorepo, que levanta Mongo + Redis + backend + worker juntos:

```bash
docker compose up -d --build
```

### Mongo

Mismo criterio: en producción usa MongoDB Atlas (o el servicio managed de tu
nube) en vez de una instancia local. Define `MONGO_URI` con la cadena de
conexión del cluster.

### Fotos de producto — Cloudinary

Las fotos de producto (subidas desde Inventario) se suben a Cloudinary
(`backend/src/utils/cloudinary.ts`) — no se guardan en disco. Define
`CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY` y `CLOUDINARY_API_SECRET`
en tu `.env` (los tres salen del dashboard de Cloudinary, pestaña "API
Keys") antes de ir a producción; sin ellos, subir una foto responde un
error 502 en vez de fallar silenciosamente.

Esto **no aplica** a las compras del cajero — ese flujo nunca tuvo subida
de foto de recibo implementada (solo producto/cantidad/monto), así que no
hay nada de "recibos" que migrar ahí. Las fotos de producto subidas ANTES
de esta migración (si las hay) siguen en `backend/uploads/` en disco —
`app.ts` sigue sirviendo ese directorio solo por compatibilidad con esas;
en plataformas de filesystem efímero (Railway, Render, Heroku) esas fotos
viejas se pierden en cada redeploy, pero cualquier foto nueva subida vía
Cloudinary no tiene ese problema. Ver punto 41 de `backend/CLAUDE.md`
para el detalle completo de la migración.

### El worker DIAN es un proceso aparte

`npm run worker:dian` (o `node dist/workers/dianWorker.js` en producción) debe
desplegarse como un **proceso independiente** del API — no corre dentro del
mismo proceso de Express. En plataformas como Railway/Render esto normalmente
significa crear un segundo "service" apuntando al mismo repo pero con otro
comando de arranque. El `docker-compose.yml` ya lo modela así (`backend` y
`dian-worker` como servicios separados que comparten Redis/Mongo).

Si usas la sincronización con Google Sheets, `npm run worker:sheets` (o
`node dist/workers/sheetsWorker.js`) sigue el mismo principio — un tercer
servicio separado (`sheets-worker` en `docker-compose.yml`), que solo
necesita Redis (no Mongo) y la variable `GOOGLE_SHEETS_WEBHOOK_URL`.

### Variables de entorno en producción

Nunca subas tu `.env` real al repositorio. Configura las variables directamente
en el panel de tu proveedor (Railway, Render, Fly.io, un VPS con systemd, etc.),
usando `.env.example` como referencia de qué necesitas definir.

## Estado de la integración DIAN

Por decisión del equipo, la integración con el PTA (Factus/Alegra/Siigo) está
**mockeada** (`backend/src/services/dianService.ts`). Simula latencia, un 5%
de fallos aleatorios (para probar el backoff exponencial) y genera un
CUFE/QR ficticio. Para conectar un proveedor real:

1. Define `DIAN_PROVIDER`, `DIAN_API_URL` y `DIAN_API_KEY` en `.env`.
2. Implementa la llamada HTTP real dentro de `DianService.emit()`,
   manteniendo la misma interfaz (`DianEmissionResult`) para no tocar el
   resto del sistema (worker, controladores, modelos).

## Qué está implementado vs. pendiente

**Implementado (funcional):**
- Modelos de datos completos (Branch, User, Product, Sale, CashClosure, Supply,
  AccountPayable/Receivable, Expense, Purchase) según el PRD + los cambios
  de arquitectura de roles.
- **Login unificado con diferenciación de rol** (cajero PIN+sede vs. admin
  correo+contraseña) en una sola app.
- **Autenticación por cookies httpOnly + axios**: el token nunca toca
  `localStorage` ni el JS del navegador; viaja en una cookie que el backend
  pone y limpia. Guardas de ruta asíncronas (`GET /auth/me`) en vez de
  chequeo síncrono de localStorage.
- **Zona de cajero con 3 pestañas** (Caja / Facturas / Compras) dentro de
  `admin-frontend`, con guarda de ruta y token propios.
- Asignación explícita de sede al crear un cajero desde `/personal`.
- **Flujo de venta POS 100% en línea**: toda venta (panel admin o caja del
  cajero) valida stock disponible y lo descuenta al confirmarse — si no hay
  conexión o no alcanza el stock, la venta no se registra y el cajero ve el
  error con la orden intacta en pantalla para reintentar (ya no hay cola
  offline para ventas nuevas — ver "Pendiente" más abajo sobre por qué se
  quitó). El grid de productos del cajero además solo ofrece productos con
  stock disponible en su sede.
- Emisión DIAN asíncrona vía BullMQ con reintentos por backoff exponencial
  + job de reconciliación automático.
- Regla de tope de consumidor final (REQ-10) con captura obligatoria de datos
  del comprador.
- Arqueo de caja con Reporte X/Z (REQ-08) — el cierre de turno muestra un
  resumen de ventas/gastos del turno (base + ventas por método de pago −
  gastos de caja menor = esperado) antes de que el cajero declare lo
  contado, además de una verificación de stock.
- Panel admin con las 9 secciones de navegación (REQ-06) + gestión de Sedes,
  dashboard con KPIs y gráficos, CRUD de productos/personal/sedes, CPP/CPC,
  gastos y vista consolidada de Compras (registradas por los cajeros).
- Modal de producto en Inventario con carga de foto optimizada a WebP — la
  MISMA URL que consume el grid de productos del cajero.
- **Gestión directa de stock por sede, solo ADMIN** (`ManagedStockModal`,
  botón "Gestionar stock" en Inventario, ver punto 60 de admin-frontend/
  CLAUDE.md): fija el stock exacto de un producto en cada sede (no lo
  suma, como sí hace el top-up existente) — pensado para declarar stock
  físico que ya existía antes de usar el sistema, o corregir un conteo sin
  simular una compra. Al crear un producto nuevo, el mismo componente
  aparece opcionalmente con el link "+ Agregar inventario inicial" bajo el
  precio — el stock elegido ahí se guarda junto con el producto al
  presionar "Crear producto", no como un paso aparte después.
- Todos los controladores envueltos en `asyncHandler`: un fallo async ya no
  deja una petición colgada sin respuesta.
- **Paginación desde el backend** en Inventario, Personal, Sedes, Compras,
  Ventas y Gastos (`page`, `pageSize`, `includeInactive` donde aplica), cada
  una con botones Anterior/Siguiente (Inventario/Personal/Sedes además con
  checkbox "Mostrar inactivos/as"; Compras y Gastos con un `totalAmount`
  agregado sobre todo el filtro, no solo la página actual).
- **Filtros del panel admin adaptados a celular** en las 7 páginas
  paginadas (Ventas, Inventario, Compras, Gastos, Personal, Sedes,
  FinanzasCaja): `pageSize` más chico en pantallas angostas, y solo el
  buscador (si la página tiene uno) queda visible junto a un botón "Más
  filtros" que abre el resto en un modal — el resto de los filtros no
  desaparece, solo se colapsa (ver punto 63 de admin-frontend/CLAUDE.md).
- **Registrar una venta manualmente desde el panel admin** (`/ventas` →
  "+ Agregar venta"): elige sede, agrega uno o más productos (solo se
  ofrecen los que tienen stock en esa sede), método de pago, categoría
  (Regular/Especial — una etiqueta manual, sin lógica automática asociada)
  y datos de cliente opcionales, y muestra un recibo imprimible (con el
  logo de la marca) al terminar.
- **Editar y cancelar ventas** desde `/ventas` (menú de 3 puntos por fila):
  editar es solo metadata (método de pago, canal, categoría, cliente);
  cancelar es un soft-delete (queda `CANCELLED`, no se borra) que restaura
  el stock si la venta lo había descontado.
- **Registrar, editar y eliminar compras desde el panel admin** (`/compras`
  → "+ Nueva compra"): un formulario cubre proveedor + múltiples sedes, y
  dentro de cada sede múltiples productos, en una sola orden. Editar/
  eliminar valida que el stock siga disponible antes de tocarlo.
- **Restricciones adicionales para GERENTE de sede**: no ve "Sedes" ni
  "Personal" en el menú (ni por URL directa), y solo puede registrar
  compras para su propia sede.
- **Sincronización en tiempo real con Google Sheets** (opcional): inventario,
  ventas, compras, gastos y cierres de caja se reflejan en una hoja de
  Google vía un worker separado — ver
  `docs/GOOGLE_SHEETS_INTEGRATION.md`.
- Mostrar/ocultar contraseña en el login de administrador.
- Panel de sesión rediseñado en la barra lateral (avatar, nombre y rol del
  usuario actual sobre el botón de cerrar sesión).
- **Sección Finanzas** (`/finanzas`), visible para ADMIN y GERENTE:
  - **Caja**: CRUD manual de aperturas/cierres de turno de caja (base de
    efectivo, base de Nequi, declarado, diferencia, etc.) — el admin/
    gerente puede agregar, editar o eliminar cualquier registro,
    independiente del arqueo automático que ya hace el cajero desde el POS.
  - **Reportes**: exporta un reporte financiero (resumen + detalle de
    ventas, compras y gastos) para un rango de fechas elegido, como Excel
    o PDF — ambos archivos generados en el backend.

**Pendiente / siguientes pasos sugeridos:**
- Integración real con el PTA elegido (Factus/Alegra/Siigo).
- Integración real de delivery (Hubster/Deliverect) — actualmente no hay
  webhooks receptores implementados.
- KDS (Kitchen Display System) vía WebSockets — no implementado. La
  impresión térmica de recibos sí está implementada (ver `print-server/`
  y `docs/THERMAL_PRINTER_INTEGRATION.md`).
- Kardex de inventario con descuento automático por receta (BOM) al vender
  — el modelo `recipe` existe en `Product` pero no se descuenta stock de
  insumos aún (distinto del descuento de producto terminado, que sí existe).
- La sincronización con Google Sheets no tiene job de reconciliación (a
  diferencia de DIAN) ni se ha probado bajo carga sostenida — ver
  `docs/GOOGLE_SHEETS_INTEGRATION.md`.
- ~~Migración de fotos a object storage~~ — hecho para fotos de producto
  (Cloudinary, ver punto 41 de `backend/CLAUDE.md`). No existe hoy subida
  de foto de recibo en el código (ver la corrección en ese mismo punto).
- Tests automatizados (unitarios e integración).
- Decidir el destino final de `pos-frontend` (ver su propio README): hoy es
  legado; si nadie lo necesita como terminal aislada, se puede retirar del
  monorepo en una futura limpieza. No fue actualizado con ninguno de los
  cambios recientes (venta online-only, Google Sheets, restricciones de
  gerente, etc.) — la fuente de verdad sigue siendo `admin-frontend`.
