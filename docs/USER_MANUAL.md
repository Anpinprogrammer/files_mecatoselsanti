# Manual de Usuario — Mecatos el Santi

Guía paso a paso para usar el sistema de **punto de venta (Caja)** y el
**panel de administración**. Está escrita para personas que no son técnicas:
cajeros, gerentes de sede y administradores.

> **¿Cómo leer este manual?**
> Cada sección indica para quién es:
> - 🧾 **Cajero** — quien atiende la caja.
> - 🛠️ **Administrador / Gerente** — quien revisa ventas, inventario y finanzas.
>
> Los nombres de botones y pantallas aparecen **en negrita**, tal como los ves en el sistema.

---

## Contenido

1. [Introducción e inicio de sesión](#1-introducción-e-inicio-de-sesión)
2. [Caja y ventas — Cajero](#2-caja-y-ventas--cajero)
3. [Panel de administración y finanzas — Administrador](#3-panel-de-administración-y-finanzas--administrador)
4. [Gestión de inventario](#4-gestión-de-inventario)
5. [Solución de problemas frecuentes](#5-solución-de-problemas-frecuentes)

---

## 1. Introducción e inicio de sesión

### 1.1 ¿Qué es este sistema?

Es una sola aplicación web con dos zonas. El sistema te lleva a la que te
corresponde según cómo inicies sesión:

| Si eres… | Entras con… | Ves… |
|---|---|---|
| **Cajero** | Tu sede + PIN de 4 dígitos | La zona de **Caja** (pestañas Caja, Facturas y Compras) |
| **Administrador** o **Gerente de sede** | Correo y contraseña | El **panel de administración** (Dashboard, Inventario, Ventas, Finanzas…) |

> ⚠️ **La zona de Caja solo funciona en computador o tableta.** Si un cajero
> entra desde un celular, el sistema muestra un aviso pidiendo usar un
> computador y solo permite **Cerrar sesión**. Esto es a propósito.
> El panel de administración **sí** se puede usar desde el celular.

### 1.2 Cómo entrar al sistema

1. Abre el navegador (Chrome, Edge o Safari) en el computador o celular.
2. Escribe la dirección del sistema: **[DIRECCIÓN DEL SISTEMA — pídesela al administrador]**.
3. Verás la **pantalla de entrada** con dos opciones grandes: **Soy Cajero** y **Soy Administrador**.

![Pantalla de entrada con las opciones Soy Cajero y Soy Administrador](./assets/manual/login-elegir-rol.png)

> 💡 **Consejo:** guarda la dirección en favoritos para no tener que escribirla cada vez.

### 1.3 Entrar como cajero (con PIN)

1. Toca **Soy Cajero** (dice "Acceso con PIN").
2. En **Selecciona tu sede**, toca la sede donde vas a trabajar.
3. Escribe tu **PIN de 4 dígitos** en el teclado de la pantalla.
4. Entrarás directamente a la zona de **Caja**.

![Selección de sede para el cajero](./assets/manual/login-cajero-sede.png)
![Teclado para escribir el PIN de 4 dígitos](./assets/manual/login-cajero-pin.png)

- Si el PIN es incorrecto, el sistema lo indica y puedes intentarlo de nuevo.
- Para volver atrás, usa el botón **Volver**.
- Si **olvidaste tu PIN**, pídele al administrador que te lo asigne de nuevo
  (los cajeros no tienen correo, por eso no pueden recuperarlo solos).

### 1.4 Entrar como administrador o gerente

1. Toca **Soy Administrador**.
2. Escribe tu **Correo** y tu **Contraseña**.
3. Toca **Ingresar**.
4. Entrarás al **Dashboard** (el resumen del negocio).

![Pantalla de ingreso del administrador](./assets/manual/login-admin.png)

### 1.5 Si olvidaste tu contraseña (administrador o gerente)

1. En la pantalla de ingreso del administrador, toca **¿Olvidaste tu contraseña?**
2. En **Recuperar contraseña**, escribe tu correo y toca **Enviar enlace**.
3. Revisa tu correo (mira también la carpeta de **spam** o correo no deseado).
   El sistema siempre muestra el mismo mensaje, exista o no el correo, por seguridad.
4. Abre el enlace del correo. Verás la pantalla **Nueva contraseña**.
5. Escribe la nueva contraseña, repítela en **Confirmar contraseña** y toca **Guardar contraseña**.
6. Vuelve a la pantalla de entrada e ingresa con la contraseña nueva.

![Pantalla Recuperar contraseña](./assets/manual/recuperar-contrasena.png)
![Pantalla Nueva contraseña](./assets/manual/nueva-contrasena.png)

> ⚠️ El enlace del correo **sirve una sola vez** y vence pasado un tiempo.
> Si dice **Enlace inválido**, repite el proceso desde el paso 1.

### 1.6 Cerrar sesión

Usa el botón **Salir** (arriba a la derecha en la zona de Caja) o el menú del
panel de administración. Hazlo siempre que dejes un computador compartido.

---

## 2. Caja y ventas — Cajero

> 🧾 Esta sección es para el **cajero**.

### 2.1 Antes de vender: abrir el turno

No puedes vender sin un **turno abierto**. Si entras y no hay uno, verás un
aviso **Turno no iniciado** con el botón **Iniciar turno**. (Las pestañas y el botón **Salir** siguen
funcionando; solo falta abrir el turno.)

![Aviso de turno requerido con el botón Iniciar turno](./assets/manual/turno-requerido.png)

**Para abrir el turno:**

1. Toca **Iniciar turno**. Se abre **Apertura de turno**.
2. Escribe la **Base inicial en efectivo** (el dinero con el que empieza la caja).
3. Escribe la **Base inicial en cuentas (Nequi)**, si aplica.
4. **Verifica el inventario**: verás una tabla con los productos de tu sede y su cantidad.
   En cada fila toca:
   - **✓** si la cantidad de la pantalla **coincide** con lo que hay físicamente.
   - **✗** si **no coincide**: escribe la cantidad real y confírmala con el ✓ pequeño (o Enter).
   Todas las filas deben quedar revisadas para poder continuar.
5. Toca **Abrir**. Verás el aviso "Turno abierto correctamente".

![Apertura de turno con verificación de inventario](./assets/manual/turno-apertura.png)

> 💡 La primera vez que entres, el sistema te muestra un **recorrido guiado**
> con las partes de la pantalla. Puedes usar **Omitir** para saltarlo.

### 2.2 Conociendo la pantalla de Caja

La pantalla de Caja tiene **tres zonas**:

![Vista general de la pantalla de Caja con sus tres zonas](./assets/manual/caja-vista-general.png)

| Zona | Dónde está | Para qué sirve |
|---|---|---|
| **Categorías y productos** | Izquierda | Elegir lo que el cliente pide. Arriba hay pestañas por categoría y un **buscador** (por nombre o código). Los productos se muestran de 9 en 9; usa **Anterior** / **Siguiente** para cambiar de página. |
| **Orden Actual** | Centro | La lista de lo que va a comprar el cliente, con cantidades y total. |
| **Cobro** | Derecha | Método de pago, **Pedido DiDi**, botones **Gasto** y **Merma**, y el botón grande **Cobrar**. |

En la parte de arriba también están las pestañas **Caja**, **Facturas** y
**Compras**, el botón **Finalizar turno** (solo aparece con un turno abierto) y **Salir**.

### 2.3 Registrar una venta paso a paso

**Paso 1 — Elegir los productos**

1. Toca la **categoría** o usa el **buscador**.
2. Toca el producto: se agrega a la **Orden Actual**.
3. Un producto que ya está en la orden aparece **deshabilitado** (gris) en la
   cuadrícula. Para cambiar la cantidad, hazlo en la Orden Actual.

**Paso 2 — Ajustar cantidades**

En la **Orden Actual**:
- **+** y **−** suben o bajan la cantidad.
- **Toca el número** para escribir la cantidad directamente.
- **✕** quita el producto de la orden.

![Orden Actual con botones de más, menos y quitar](./assets/manual/caja-orden-actual.png)

> ⚠️ Si pides más unidades de las que hay en inventario, el sistema no deja
> registrar la venta y muestra **"No hay stock suficiente para: …"** con lo que sí hay disponible.

**Paso 3 — Elegir el método de pago**

En **Método de pago** hay tres opciones:

| Método | Cuándo usarlo |
|---|---|
| **Efectivo** | El cliente paga con billetes o monedas. |
| **Nequi** | El cliente paga por Nequi. |
| **Bancolombia** | El cliente paga por transferencia Bancolombia. |

**Si elegiste Efectivo:**
1. Escribe cuánto **Efectivo recibido** te dio el cliente, o toca **💵 Billetes y monedas**
   para ir tocando los billetes y monedas que recibes.
2. El sistema calcula las **Vueltas** (lo que debes devolver). Si sale en rojo,
   falta dinero.

![Cobro en efectivo con billetes y vueltas](./assets/manual/caja-pago-efectivo.png)

**Paso 4 — Cobrar**

1. Revisa el **Resumen del pedido** (abajo, antes del botón).
2. Toca **Cobrar $…** (el botón muestra el total).
3. Según el caso, el sistema hará una de estas preguntas:

**a) ¿Factura electrónica?** (ventas normales)
Pregunta si el cliente quiere factura electrónica:
- **No, gracias** → la venta se registra de inmediato.
- **Sí, quiero factura** → se abre **Datos del comprador**: escribe **Nombre / Razón social**,
  **Cédula / NIT** y **Correo electrónico**, y toca **Confirmar y cobrar**.

![Pregunta ¿Factura electrónica?](./assets/manual/caja-pregunta-factura.png)
![Datos del comprador](./assets/manual/caja-datos-comprador.png)

**b) Venta grande (por encima del tope de la DIAN)**
Si el total supera el tope legal (hoy **$509.000**), **los datos del comprador son
obligatorios**: el sistema abre directamente **Datos del comprador**, sin preguntar.

**Paso 5 — Confirmación**

- Verás el aviso verde **Venta registrada correctamente** arriba a la derecha.
- Si el cliente pidió factura electrónica, aparece una pantalla que dice
  **Emitiendo factura ante la DIAN…** con una barra de progreso. Espera unos
  segundos: el recibo aparece solo cuando la factura está lista. Si demora
  mucho, puedes tocar **Ver recibo de todas formas**.
- Después se abre el **recibo** de la venta (ver 2.5).

![Pantalla Emitiendo factura ante la DIAN](./assets/manual/caja-emitiendo-factura.png)

> ⚠️ **Si la venta falla** (por ejemplo, se cayó el internet), el sistema
> muestra el error y **conserva la orden en pantalla**. No pierdes lo que
> habías armado: arregla la conexión y vuelve a tocar **Cobrar**. **Toca Cobrar
> una sola vez** y espera; tocarlo varias veces seguidas puede generar confusión.

### 2.4 Pedidos DiDi Food

Los pedidos que llegan por **DiDi Food** se marcan con el interruptor
**Pedido DiDi**, encima de **Método de pago**.

1. Arma la orden normalmente.
2. **Activa el interruptor "Pedido DiDi"** (se pone naranja/encendido).
3. Al activarlo, el método de pago se fija en **Bancolombia** y **Efectivo** y
   **Nequi** quedan bloqueados. Esto es normal: DiDi no le paga a la caja en el
   momento, le **deposita el dinero a la cuenta Bancolombia** después.
4. En el **Resumen del pedido** verás "Pedido DiDi — Bancolombia (Cuenta por Cobrar)".
5. Toca **Cobrar**.

**¿Qué significa "pago pendiente"?**

- La venta **queda registrada y descuenta inventario** de inmediato, pero se
  marca como **pendiente de pago**: el dinero todavía no está en el banco.
- **No suma al efectivo de tu caja.**
- Cada **miércoles** (día de liquidación semanal de DiDi) el administrador
  revisa el extracto del banco y **confirma** los depósitos recibidos.
  Ver [3.4 Conciliación de ventas DiDi](#34-conciliación-de-ventas-didi-miércoles).

> 💡 **Tú no tienes que hacer nada más** con esas ventas. Solo asegúrate de
> activar **Pedido DiDi** en cada pedido que venga por esa aplicación; si no,
> se registraría como una venta normal y no se cruzaría con el depósito.

![Interruptor Pedido DiDi activado con método Bancolombia](./assets/manual/caja-pedido-didi.png)

### 2.5 Imprimir el recibo (impresora térmica)

Cuando termina una venta, se abre el **recibo** en pantalla. Con él puedes:

| Botón | Qué hace |
|---|---|
| **Imprimir recibo** | Manda el recibo a la **impresora térmica** de la caja. |
| **Vista previa** | Muestra en una ventana nueva cómo quedaría el ticket, sin imprimir. |
| **Cerrar** (o la **X** de la esquina) | Cierra el recibo. |

![Recibo de la venta con los botones Cerrar, Vista previa e Imprimir recibo](./assets/manual/caja-recibo.png)

**Cómo funciona:**
- El recibo **no se imprime solo**: tienes que tocar **Imprimir recibo**.
- Para imprimir directo en la térmica, el computador de la caja debe tener
  encendido el **programa de impresión** (ver [sección 5.1](#51-la-impresora-térmica-no-responde)).
- Si ese programa no está disponible, el sistema abre la **ventana de impresión
  del navegador** como respaldo. Ahí puedes elegir la impresora y continuar.
- Puedes volver a ver e imprimir cualquier recibo de tu turno desde la pestaña
  **Facturas** → **Ver recibo**.

### 2.6 Gastos de caja menor (**Gasto**)

Úsalo cuando **sale efectivo de la caja** para un gasto pequeño (por ejemplo,
comprar bolsas).

1. En el panel de cobro, toca **Gasto**. Se abre **Gasto caja menor**.
2. Escribe el **Concepto** (ej. "Compra de bolsas").
3. Escribe el **Monto**.
4. Toca **Registrar**.

![Formulario Gasto caja menor](./assets/manual/caja-gasto.png)

El gasto **se resta del efectivo esperado** al cerrar el turno, así el cuadre queda correcto.

### 2.7 Mermas — productos dañados o consumidos (**Merma**)

Úsalo cuando **baja el inventario sin que haya una venta**: producto dañado,
vencido, o consumido por un empleado.

1. En el panel de cobro, toca **Merma**. Se abre **Registrar merma de stock**.
2. Elige el **Producto** (verás cuánto stock disponible tiene).
3. Escribe la **Cantidad**.
4. Elige el **Motivo**:
   - **Producto dañado / vencido**
   - **Consumo interno (empleado)**
   - **Otro**
5. Si quieres, agrega una **Nota (opcional)**.
6. Toca **Registrar**.

![Formulario Registrar merma de stock](./assets/manual/caja-merma.png)

> ⚠️ No puedes registrar una merma mayor a lo que hay en inventario.
> Una merma **no es una venta ni un gasto de dinero**: solo baja el inventario.

### 2.8 Compras desde la pestaña Compras (resumen)

Si compras insumos o mercancía con dinero de la caja, regístralo en la pestaña
**Compras** → **+ Nueva compra**: escribe el **Proveedor**, el **Concepto** (opcional) y,
por cada producto, el **Monto pagado** y la **Cantidad**. Al tocar **Registrar compra**
el inventario de tu sede **sube** y el dinero **se resta del efectivo esperado**.

En cada fila puedes buscar el producto escribiendo su nombre o código.

### 2.9 Cerrar el turno (**Finalizar turno**)

Al terminar tu jornada:

1. Toca el botón **Finalizar turno** (arriba, en el centro). Se abre **Cierre de turno**.
2. A la izquierda verás el **Resumen del turno**:
   - Lo vendido por método de pago (**Efectivo**, **Nequi**, **Apps**).
   - Cuánto **debería haber** en efectivo: base inicial + ventas en efectivo
     − compras − gastos.
3. Cuenta tu caja y escribe:
   - **Efectivo contado**
   - **Saldo en cuentas (Nequi)**
4. A la derecha, **verifica el inventario** fila por fila con **✓** (coincide) o **✗**
   (no coincide, escribe la cantidad real), igual que al abrir.
5. Toca **Cerrar turno**.
6. Verás el resultado: **Efectivo declarado**, **Efectivo esperado** y la
   **Diferencia en efectivo** (igual para Nequi).

![Cierre de turno con resumen e inventario](./assets/manual/turno-cierre.png)
![Resultado del cierre con diferencias](./assets/manual/turno-resultado.png)

> 💡 **Diferencia = 0** significa que todo cuadra. Si hay una diferencia, el
> administrador la ve en Finanzas y puede revisar el detalle del turno.
> No es un castigo: sirve para detectar errores a tiempo.

Puedes cerrar cualquier ventana de turno con la **✕** de la esquina o con **Cancelar**.

---

## 3. Panel de administración y finanzas — Administrador

> 🛠️ Esta sección es para **administradores** y **gerentes de sede**.
> Los gerentes ven un menú más corto: por ejemplo, **Sedes** y **Personal** son
> solo para el administrador.

### 3.1 Moverse por el panel

- El **menú lateral** tiene: **Dashboard**, **Sedes**, **Inventario**, **Compras**,
  **Ventas**, **Gastos**, **Personal** y **Finanzas** (Caja, Reportes, Cuentas por Cobrar,
  Cuentas por Pagar, Config. DIAN).
- La **barra superior** tiene el **selector de sede**: todo lo que ves
  (ventas, inventario, gráficas) se filtra por la sede elegida. Déjalo en "todas"
  para ver el total del negocio.
- En el celular el menú se abre con el botón de las tres rayas.

![Panel de administración con menú lateral y selector de sede](./assets/manual/admin-menu.png)

### 3.2 Entender el Dashboard

El **Dashboard** es el resumen del negocio. Arriba eliges el **periodo**:
**Hoy**, **Ayer**, **Esta semana** o **Este mes**.

Cuatro tarjetas muestran lo esencial:

| Tarjeta | Qué significa |
|---|---|
| **Ventas** | Total vendido en el periodo. |
| **Compras** | Dinero gastado en compras de mercancía e insumos. |
| **Gastos** | Dinero gastado en gastos operativos (arriendo, servicios, caja menor…). |
| **Rentabilidad** | **Ventas − Compras − Gastos**. Es una guía rápida de cuánto le queda al negocio. |

![Tarjetas de Ventas, Compras, Gastos y Rentabilidad](./assets/manual/admin-dashboard-kpis.png)

Más abajo hay gráficas como **Top 5 productos**, **Gastos por categoría** y métodos de pago.

> 💡 La **Rentabilidad** es una referencia sencilla, no un balance contable.

### 3.3 Leer la alerta de **Stock Crítico**

El recuadro **Stock Crítico** del Dashboard te avisa qué productos se están acabando.

- Arriba a la derecha aparece una etiqueta roja como **3 productos en riesgo**.
- Si todo está bien, verás **"Todo el inventario está al día"** con un visto verde.
- Cada producto muestra su nombre, su código (SKU) y una etiqueta:
  - **Sin stock** (roja): se agotó por completo.
  - **5 / 10 unds** (naranja): quedan 5 unidades y el mínimo configurado era 10.
- El stock crítico se calcula con la **sede elegida** arriba (o el total de todas).
  **No depende** del periodo Hoy/Ayer/Semana/Mes: es la foto del inventario **de este momento**.
- Toca **Ir a Inventario** para abrir el inventario ya filtrado solo con esos productos.

![Widget Stock Crítico con productos en riesgo](./assets/manual/admin-stock-critico.png)

> 💡 Un producto en **0 unidades** siempre aparece como crítico. Para que un
> producto **avise antes de agotarse**, edítalo en **Inventario** y llena su
> **Stock mínimo** (0 significa "sin alerta").

### 3.4 Conciliación de ventas DiDi (miércoles)

**¿Qué es?** Cada miércoles DiDi deposita en Bancolombia lo vendido en la
semana. Las ventas de DiDi quedaron registradas como **pendientes de pago**; ahora
debes **confirmar** las que sí llegaron al banco.

> ℹ️ **Dónde se hace:** en la página **Ventas** (no en Cuentas por Cobrar).
> La página **Cuentas por Cobrar** del menú Finanzas es otra herramienta: sirve para
> anotar **créditos manuales** a clientes (botón **+ Nuevo crédito**). Aunque el
> resumen de la caja diga "Cuenta por Cobrar" al vender por DiDi, la confirmación
> semanal se hace desde **Ventas**.

**Paso a paso:**

1. Los **miércoles**, al abrir **Ventas**, aparece una franja amarilla:
   **"Es miércoles de liquidación DiDi. Revisa tu extracto bancario y confirma el
   depósito — hay N ventas pendientes."** (el sistema puede abrir la lista solo).
   Otros días del año puedes abrirla con el botón **Pendientes DiDi/Rappi**.
2. Toca **Ver pendientes**. Se abre **Ventas DiDi/Rappi pendientes de pago** con una tabla:
   **Fecha**, **Sede**, **Cajero** y **Total**.
3. **Abre tu extracto de Bancolombia** y compara: ¿cuánto depositó DiDi?
4. En la tabla, **marca las ventas que sí están incluidas en el depósito**
   (casilla al lado de cada una). Con la casilla de arriba, **Seleccionar todas**,
   marcas todas.
5. Mira el contador de abajo: **"N de M seleccionadas"** y el **total en pesos**.
   **Ese total debe coincidir con el depósito del banco.**
6. Si quieres, escribe el **Número de comprobante / transferencia** (opcional).
7. Toca **Confirmar N ventas** (el botón dice cuántas seleccionaste).
8. Verás un mensaje de éxito. Si alguna no se pudo confirmar (por ejemplo, otra
   persona ya la había confirmado), el sistema te dice cuántas sí y cuántas no.

![Franja amarilla del miércoles de liquidación DiDi en Ventas](./assets/manual/admin-didi-banner.png)
![Lista de ventas DiDi/Rappi pendientes con casillas y botón Confirmar](./assets/manual/admin-didi-modal.png)

> ⚠️ **Confirma solo lo que ya viste en el banco.** El sistema no tiene forma
> de comprobarlo por ti. Las ventas sin confirmar siguen como pendientes y no
> se cuentan como dinero recibido.
>
> 💡 Para confirmar **una sola** venta, en la tabla de Ventas usa el menú de acciones
> de esa fila y elige **Confirmar Pago**.

### 3.5 Otras pantallas de Finanzas (resumen)

- **Finanzas → Caja:** cierres de turno de los cajeros. Cada fila tiene un botón
  **Ver** con el detalle del turno (ventas, compras, gastos, diferencias).
- **Finanzas → Reportes:** reportes exportables a Excel y PDF por rango de fechas.
- **Gastos** y **Compras:** listas de lo gastado, con botón para crear y menú de acciones
  para **Editar** o **Eliminar**.
- **Ventas:** todas las ventas, con filtros y menú de acciones por venta.

---

## 4. Gestión de inventario

### 4.1 Ver el stock actual por sede

1. En el menú lateral, entra a **Inventario**.
2. En la barra superior, elige la **sede**. La columna **Stock** muestra las unidades
   **de esa sede**; con "todas las sedes" muestra el total.
3. La tabla muestra **Foto**, **Nombre**, **SKU**, **Categoría**, **Precio**, **Stock**,
   **Mín.** (stock mínimo), **Estado** y **Acciones**.
4. Usa el **buscador** (por nombre o SKU). También puedes mostrar productos
   inactivos o solo los de **stock bajo**. En celular, esos filtros están en
   el botón de **más filtros**.

![Pantalla de Inventario con stock por sede](./assets/manual/inventario-lista.png)

Desde aquí también puedes crear un producto con **+ Nuevo producto** y, desde el menú
de **Acciones** de cada fila, editarlo, ajustar su stock o desactivarlo.

> 💡 Cuando **compras** mercancía (página **Compras**), el stock **sube** solo.
> Cuando **vendes** en Caja, **baja** solo.

### 4.2 Corregir el inventario de muchos productos a la vez (**Carga Masiva**)

Sirve para **corregir el conteo** de muchos productos en varias sedes en una sola
pantalla (por ejemplo, después de un inventario físico).

> ⚠️ Solo el **Administrador** ve este botón (el gerente no). Además, solo aparece en
> **computador**: en el celular la tabla sería demasiado ancha.
>
> ⚠️ **Importante:** lo que escribes es la **cantidad exacta que hay**, no
> cuánto sumar. Si escribes 20, el stock queda en 20 (no "+20").
> Para *sumar* mercancía que llegó, usa **Compras**.

**Paso a paso:**

1. En **Inventario**, toca **Carga Masiva** (arriba a la derecha).
   Se abre la ventana **Asignación Masiva de Inventario**.
2. Usa la barra **Buscar por nombre o SKU…** para encontrar un producto.
3. Verás una **matriz**: cada **fila es un producto** y cada **columna es una sede**.
   Cada casilla muestra la cantidad actual.
4. **Escribe la cantidad real** en las casillas que quieras corregir.
   Las casillas que cambies se marcan de otro color.
5. Abajo verás el contador **"N celda(s) modificada(s)"** (o "Sin cambios todavía").
6. Toca **Guardar Cambios**. Solo se guardan las casillas que tocaste;
   las demás no se alteran.
7. Para salir sin guardar, toca **Cancelar**.

![Ventana Asignación Masiva de Inventario con matriz producto por sede](./assets/manual/inventario-carga-masiva.png)

> 💡 Puedes usar el buscador entre ediciones: tus cambios se conservan aunque
> cambie la lista visible.

---

## 5. Solución de problemas frecuentes

### 5.1 La impresora térmica no responde

Para imprimir directo en la impresora térmica, el computador de la caja necesita
tener **encendido el programa de impresión** (un programa pequeño que corre en segundo
plano, llamado internamente *print-server*).

**Revisa en este orden:**

1. **La impresora:** ¿está **encendida**, con **papel** y con el **cable USB** bien conectado?
2. **El programa de impresión:** debe estar **ejecutándose en ese computador**.
   Si no lo está, pídele a la persona de soporte técnico (quien instaló la impresora)
   que lo **inicie de nuevo**. Cuando el computador se reinicia, hay que volver
   a iniciarlo.
3. **Que no esté ocupada:** si otro programa está usando la impresora, ciérralo
   y vuelve a probar.
4. **Prueba de respaldo:** toca **Imprimir recibo** otra vez. Si el programa no responde, el
   sistema abre la **ventana de impresión del navegador**; elige la impresora ahí
   e imprime desde esa ventana. **Nunca pierdes el recibo.**
5. **Vista previa:** el botón **Vista previa** sirve para ver cómo quedaría el ticket.
   Si esta tampoco funciona, confirma que el programa de impresión está encendido.
6. Si aparece un mensaje de bloqueo de ventana, permite las **ventanas emergentes**
   para el sitio en el navegador.

> ℹ️ El resto del sistema (vender, cobrar, cerrar turno) funciona igual aunque
> el programa de impresión esté apagado.

### 5.2 Internet lento o se cae la conexión

- **Las ventas solo se registran con internet.** El sistema ya no guarda ventas
  "para después". Si la conexión falla al cobrar, verás un mensaje de error y
  **la orden sigue en pantalla**. Espera a que vuelva la conexión y toca **Cobrar** de nuevo.
- Toca **Cobrar una sola vez** y espera unos segundos. Al registrar la venta puede
  tardar un poco más si hay factura electrónica (hasta unos 20 segundos).
- Si un producto o pantalla se queda cargando, espera un momento y **recarga la página**
  (tecla **F5** o el botón de recargar del navegador). No perderás lo ya registrado.
- Si el sistema te devuelve a la pantalla de entrada, tu sesión venció: vuelve
  a iniciar sesión (cajero: sede + PIN).

### 5.3 El inicio de sesión tarda o no carga

- **La primera vez del día puede tardar hasta un minuto.** El servidor "se
  duerme" cuando nadie lo usa y necesita despertarse. Espera con paciencia
  y no toques el botón muchas veces.
- Si después de **1–2 minutos** no carga, recarga la página (F5) e inténtalo otra vez.
- **PIN incorrecto:** verifica que elegiste **tu sede**; el PIN solo funciona en tu sede.
- **Administrador que olvidó la contraseña:** usa **¿Olvidaste tu contraseña?**
  ([sección 1.5](#15-si-olvidaste-tu-contraseña-administrador-o-gerente)).
- **Cajero que olvidó el PIN:** pídele al administrador que se lo asigne de nuevo.
- Si nada funciona, revisa que el computador tenga internet abriendo cualquier
  otra página.

### 5.4 Mensajes frecuentes

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| **No hay stock suficiente para…** | Pediste más unidades de las que hay. | Reduce la cantidad, o pide al administrador corregir el inventario. |
| **Turno no iniciado** | No hay un turno abierto. | Toca **Iniciar turno** y ábrelo. |
| **Enlace inválido** (recuperar contraseña) | El enlace ya se usó o venció. | Pide uno nuevo con **¿Olvidaste tu contraseña?**. |
| Factura "en proceso de validación DIAN" | La factura electrónica aún se está emitiendo. | Espera; se completa sola. |
| Pantalla "usa un computador" | Un cajero entró desde el celular. | Usa el computador de la caja. |

### 5.5 ¿A quién pedir ayuda?

- **Problemas de la caja, PIN o turno:** el administrador o gerente de tu sede.
- **Impresora o computador:** la persona de soporte técnico que instaló el equipo.
- **Errores del sistema que se repiten:** anota el mensaje exacto (o toma una
  foto de la pantalla) y avisa al administrador.

---

*Este manual describe el sistema tal como funciona hoy. Si un botón o
pantalla cambia de nombre, esta guía se actualiza junto con el sistema.*
