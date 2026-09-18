# Thermal Printer Integration (ESC/POS Local Server) — Mecatos el Santi

Este documento describe la impresión térmica directa de recibos,
implementada para el panel admin (`Ventas.tsx` → "Ver recibo") y el
cajero (`Facturas.tsx` → "Ver recibo", y automáticamente al confirmar una
venta en Caja). Refleja lo que **realmente existe** en el repo — no es
solo un plan.

---

## 1. Resumen y flujo

1. **Venta completada / recibo abierto:** el modal de recibo
   (`components/SaleReceipt.tsx` en el panel admin,
   `cajero/components/SaleReceipt.tsx` en el cajero — dos copias
   intencionales, ver punto 12 de `CLAUDE.md`) muestra "Vista previa",
   "Imprimir recibo" y "Cerrar".
2. **Impresión a pedido, nunca automática:** `printThermalReceipt` solo
   se llama dentro del `onClick` del botón "Imprimir recibo". Nada se
   imprime solo al abrir el modal. "Vista previa" (`previewThermalReceipt`,
   ver sección 8) tampoco imprime nada — solo abre una pestaña con una
   aproximación HTML del ticket, útil quien no tiene la impresora física a
   la mano en ese momento.
3. **Print-server local (`print-server/`):** un servicio Express aparte
   (no es parte de `backend/`) que corre en `http://localhost:4001`, en
   la MISMA máquina física que tiene la impresora térmica USB conectada,
   y manda los comandos ESC/POS.
4. **Fallback automático:** si el print-server no responde (apagado, no
   instalado en esa máquina, o sin impresora conectada), el frontend cae
   a `window.print()` — el mismo flujo de recibo imprimible por navegador
   que ya existía antes de esto (ver punto 19 de `CLAUDE.md`). El cajero
   nunca ve un error: en el peor caso, simplemente se abre el diálogo de
   impresión del navegador en vez de mandar directo a la térmica.

---

## 2. Por qué `print-server/` es un paquete aparte (no parte de `backend/`)

La impresora térmica es un periférico físico de la terminal del cajero
(o del admin, si también imprime desde su propio equipo) — no un recurso
que el backend en la nube pueda tocar. `backend/` corre en Docker/una
nube sin acceso a hardware local de nadie. Por eso este servidor:

- Corre en `localhost` de la máquina física con la impresora.
- Usa el puerto **4001**, no 4000 — el 4000 ya es el backend real.
- No comparte código ni proceso con `backend/`; se instala y arranca por
  separado (`cd print-server && npm install && npm start`).

## 3. Librería usada: `@node-escpos/*`, NO el paquete `escpos`/`escpos-usb`

El plan original apuntaba a `escpos` + `escpos-usb` (paquetes sin
actualizar desde ~2019). **Se descartaron por dos problemas reales
encontrados al intentar instalarlos, en ese orden:**

1. `escpos-usb` declara `"usb": "*"` sin fijar versión — npm resolvió la
   última `usb` (v3.x, reescrita en Rust/napi-rs con una API distinta) y
   el código viejo de `escpos-usb` llamaba a `usb.getDeviceList()` de una
   forma que ya no existe → crash inmediato (`usb.getDeviceList is not
   a function`).
2. Al fijar `usb` a una versión vieja compatible (`1.9.2`, la única API
   que `escpos-usb` entiende), la instalación falló al compilar los
   bindings nativos (`node-gyp`/`make`) — el Makefile generado no
   escapaba correctamente la ruta del repo, que tiene espacios
   ("Proyecto Mecatos el Santi"), así que `clang++` interpretaba
   `Mecatos`, `el`, `Santi/...` como argumentos de archivo separados y
   fallaba. Esto no es un bug de este proyecto — es una limitación
   conocida de `node-gyp`/GYP con rutas que contienen espacios, y
   afectaría a **cualquier** dependencia nativa vieja instalada bajo esa
   misma ruta, no solo a esta.

**La solución:** usar la organización mantenida `@node-escpos`
(`@node-escpos/core` + `@node-escpos/usb-adapter`). Pero ojo — esto tuvo
una vuelta más: `@node-escpos/usb-adapter@0.3.1` **también** declara su
dependencia de `usb` como `"*"` (el mismo problema de fondo, en la
librería nueva). Sin fijar una versión, npm resuelve `usb@3.x`, cuyo
export de nivel superior ya no tiene `getDeviceList` — mismo síntoma que
antes (`TypeError: usb.getDeviceList is not a function`), solo que ahora
viniendo del código de `usb-adapter`, no de `escpos-usb`. La solución
real: fijar `"usb": "2.18.0"` exacto en `print-server/package.json` (la
última versión de la serie 2.x, que sí tiene `getDeviceList` donde
`usb-adapter` lo espera) — y esa versión, igual que la 3.x, se instala
con **binarios precompilados** (`node-gyp-build`/`prebuildify`), así que
sigue sin requerir compilar nada localmente y sigue evitando el problema
#2 (rutas con espacios). Ver `print-server/README.md` para el detalle
completo de este pin — no lo quites sin verificar antes que la versión
de `usb-adapter` que estés usando soporte `usb@3.x`.

**Diferencias de API relevantes al migrar `escpos` → `@node-escpos/core`:**
- El viejo `escpos` era 100% basado en callbacks; `@node-escpos/core` usa
  `Promise`/`async-await` — `printer.close()` ahora es `async` y hace su
  propio `flush()` del buffer internamente (no hace falta llamar
  `flush()` aparte).
- `printer.cut(partial?, feed?)` ahora hace su propio `feed()` antes de
  cortar (default 3 líneas) — no llames a `.feed(3)` justo antes de
  `.cut()` como en el ejemplo original, o el papel avanza el doble de lo
  esperado.
- El constructor de `Printer` recibe el adapter y un objeto de opciones
  como segundo argumento obligatorio (puede ir vacío: `new Printer(device,
  {})`).

## 4. Servidor local (`print-server/index.js`)

```bash
cd print-server
npm install   # ver print-server/README.md — el pin de usb@2.18.0
npm start     # o: npm run dev (nodemon)
```

El contrato JSON crece un poco respecto al diseño original, para que el
ticket impreso pueda replicar el mismo formato que el recibo en pantalla
(logo, dirección/teléfono de la sede, precio unitario por ítem):

```
POST http://localhost:4001/print-receipt
Body: {
  branch, branchAddress?, branchPhone?,
  invoiceId,
  items: [{ name, quantity, price, subtotal }],
  subtotal, total,
  cashier, paymentMethod
}
Respuesta éxito: { success: true, message: "..." }
Respuesta error:  { success: false, error: "..." }  (HTTP 500)
```

Internamente usa `@node-escpos/usb-adapter` (autodetecta la primera
impresora térmica USB — `new USBAdapter(vid, pid)` si hay que fijar una
específica, ver `print-server/README.md`) y `@node-escpos/core` para
construir el ticket, replicando el mismo formato que `SaleReceipt.tsx`
muestra en pantalla:

1. Logo (`assets/logo.png`, redimensionado a 384px y convertido a escala
   de grises con `sharp` UNA sola vez al arrancar — no en cada recibo,
   ver `print-server/README.md`).
2. "Mecatos el Santi" en negrita, nombre/dirección/teléfono de la sede.
3. Fila de dos columnas "Ticket: X" / fecha, y otra "Atendido por: X" /
   método de pago (`tableCustom` con dos columnas de ancho 0.5 cada una —
   mismo efecto visual que los `flex justify-between` en pantalla).
4. Tabla de ítems **con encabezado** (Producto/Cant./Precio/Subtotal — 4
   columnas, igual que la tabla en pantalla, no una versión reducida de
   2 columnas).
5. Subtotal / Total (mismas etiquetas exactas que en pantalla — sin
   impuesto: el negocio no declara/cobra IVA/INC, ver punto 65 de
   `backend/CLAUDE.md`), corte de papel y apertura de cajón.

## 5. Cliente del frontend (`printerService.ts` — dos copias)

- `admin-frontend/src/services/printerService.ts` (panel admin)
- `admin-frontend/src/cajero/services/printerService.ts` (cajero — copia
  intencional, no un import; ver punto 12 de `CLAUDE.md`: `src/cajero/`
  nunca importa nada de fuera de sí mismo)

Ambas son idénticas en contenido. Usan `fetch` (no los clientes axios
`httpClient.ts` de este proyecto) **a propósito** — es la única excepción
documentada a "no fetch en este proyecto" (ver stack en `CLAUDE.md`):
el print-server es un proceso local de la terminal, no la API en la
nube, así que no necesita cookies/credenciales ni el interceptor de
errores de axios.

```typescript
export const printThermalReceipt = async (data: PrintReceiptPayload): Promise<boolean> => {
  try {
    const response = await fetch("http://localhost:4001/print-receipt", { ... });
    if (!response.ok) throw new Error("Local printer server unreachable");
    return (await response.json()).success;
  } catch (error) {
    console.warn("Silent print failed, falling back to browser print dialog:", error);
    window.print();
    return false;
  }
};
```

## 6. Integración en `SaleReceipt.tsx` (ambas copias)

El botón "Imprimir recibo" (antes: `onClick={() => window.print()}`)
ahora llama a un `handlePrint` async que arma el payload a partir del
`sale` ya cargado en el componente y llama a `printThermalReceipt`. El
fallback a `window.print()` vive DENTRO de `printerService.ts` — el
componente no necesita manejarlo aparte.

```tsx
const [printing, setPrinting] = useState(false);

const handlePrint = async () => {
  setPrinting(true);
  try {
    await printThermalReceipt({
      branch: branch?.name || "",
      branchAddress: branch?.address,
      branchPhone: branch?.phone,
      invoiceId: String(sale._id).slice(-8).toUpperCase(),
      items: sale.items.map((it) => ({
        name: it.name,
        quantity: it.quantity,
        price: it.price,
        subtotal: it.subtotal,
      })),
      subtotal: sale.subtotal,
      total: sale.total,
      cashier: cashierName || "—",
      paymentMethod: paymentMethodLabels[sale.paymentMethod] || sale.paymentMethod,
    });
  } finally {
    setPrinting(false);
  }
};
```

- **Sin `disabled` permanente**: el cajero/admin puede pedir copias
  adicionales sin cerrar el modal — solo se deshabilita mientras
  `printing === true` (muestra "Imprimiendo...").
- **Alcance**: ambas copias de `SaleReceipt.tsx` (admin y cajero) tienen
  este botón — a diferencia de otras features de este proyecto que solo
  viven en un lado, esta se pidió explícitamente para ambas.

## 7. Verificado en vivo (sin impresora física conectada)

Se probó el flujo completo sin una impresora térmica real conectada a la
máquina de desarrollo:

- `POST /print-receipt` sin impresora conectada → `@node-escpos/usb-adapter`
  lanza `"Can not find printer"` → el servidor responde `500 { success:
  false, error: "Thermal printer not connected or busy" }` (antes, con el
  paquete viejo, esto era un crash sin manejar, no un error limpio — y
  durante el desarrollo de esto, con `usb@3.x` sin fijar, era OTRO crash
  distinto sin manejar, `usb.getDeviceList is not a function`; ver
  sección 3).
- El logo se procesó sin errores al arrancar el servidor (`sharp` +
  `Image.load`, ver sección 4) — se verificó revisando el log de arranque
  buscando el mensaje de error que `getLogoImage()` loguea si falla (no
  apareció).
- Desde el navegador real (Ventas → Ver recibo → Imprimir recibo): se
  interceptó la petición real a `localhost:4001` y se confirmó que el
  payload capturado trae `branchAddress`/`branchPhone` y `price` por
  ítem — coincide exactamente con los datos de la venta mostrada en
  pantalla. El fetch falla como se espera (sin impresora), se dispara el
  fallback, `window.print()` se llama, y el botón vuelve a su estado
  normal (no queda atascado en "Imprimiendo...").

Lo que **no** se pudo verificar en este entorno (requiere una impresora
térmica USB real conectada): que el ticket físico impreso se vea bien
formateado — en particular, que el logo se vea nítido (no borroso) al
tamaño de 384px con el dithering de `@node-escpos/core`, y que el corte
de papel y la apertura del cajón funcionen. Eso solo se puede confirmar
en la terminal real del cajero.

## 8. Vista previa sin impresora física conectada (`/preview-receipt`)

`SaleReceipt.tsx` (ambas copias) tiene un segundo botón, "Vista previa",
junto a "Imprimir recibo" — pensado para poder revisar el formato del
ticket térmico sin tener una impresora física conectada a la máquina
(útil en desarrollo, o para confirmar cambios de formato antes de ir a la
terminal real del cajero).

- **Backend**: `POST /preview-receipt` en `print-server/index.js` acepta
  exactamente el mismo body que `/print-receipt`, pero en vez de hablar
  con el USB devuelve una página HTML completa que aproxima visualmente
  el ticket — mismo logo ya procesado (`sharp`, escala de grises,
  cacheado en memoria, ver sección 4) como `<img>` embebido en base64, y
  filas `flex` que replican los mismos anchos de columna
  (`TWO_COL_WIDTHS`/`ITEM_COL_WIDTHS`, constantes compartidas con el path
  de impresión real) que usa `tableCustom` en el ESC/POS de verdad — así
  ambos endpoints no pueden desincronizarse en su layout.
- **Texto largo se trunca, no hace wrap**: las celdas de la vista previa
  usan `white-space: nowrap; overflow: hidden; text-overflow: ellipsis`
  a propósito — una impresora térmica real tampoco ajusta el texto de una
  columna fija a una segunda línea (lo trunca o desborda, según el
  modelo). Ver puntos suspensivos en la vista previa es una señal
  legítima de que ese nombre de producto/cajero es demasiado largo para
  la columna real, no un bug de la vista previa.
- **Frontend**: `previewThermalReceipt` (`printerService.ts`, ambas
  copias) hace `fetch` a `/preview-receipt`, y abre el HTML resultante en
  una pestaña nueva con `window.open("", "_blank")` +
  `document.write(html)`. A diferencia de `printThermalReceipt`, **no
  tiene fallback silencioso** — si el print-server no está corriendo,
  lanza el error tal cual, y `handlePreview` en `SaleReceipt.tsx` lo
  muestra con un `alert()`, porque el único propósito de este botón es
  justamente confirmar visualmente el resultado; fallar en silencio no
  tendría sentido acá.
- Ambos botones ("Vista previa" e "Imprimir recibo") comparten la misma
  función interna `buildReceiptPayload()` en `SaleReceipt.tsx` — no se
  duplica el armado del payload entre los dos flujos.

Verificado en este entorno (sin impresora física): se generó el HTML con
`curl`, se renderizó con Chrome en modo headless a una imagen, y se
confirmó visualmente que coincide con el recibo en pantalla (logo, datos
de sede, tabla de ítems con encabezado, totales) — incluyendo, con un
nombre de cajero y de producto deliberadamente largos, que el truncado
con "…" funciona como se espera en vez de partir la fila a dos líneas.

## 9. Troubleshooting

Ver `print-server/README.md` para: el pin de `usb@2.18.0` (por qué existe
y qué hacer si algo lo rompe), cómo fijar un vendor/product id específico
si hay más de un dispositivo USB conectado, cómo cambiar el logo, y qué
significa cada error común.
