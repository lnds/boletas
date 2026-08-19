# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es

Calculadora de boletas de honorarios (Chile) en un **único archivo autocontenido**:
`calculadora-boletas.html`. No hay build, dependencias, servidor ni framework — se abre
con `file://` en el navegador. Todo cambio de estilo, marcado o lógica va inline en ese
mismo archivo (`<style>`, el cuerpo del `<div class="wrap">` y `<script>`).

## Comandos

```bash
# Abrir la página
open calculadora-boletas.html

# Chequear la sintaxis del script embebido (no hay suite de tests)
sed -n '/^<script>$/,/^<\/script>$/p' calculadora-boletas.html | sed '1d;$d' > /tmp/chk.js
node --check /tmp/chk.js

# Verificar la vista de impresión sin abrir el diálogo de impresión
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu \
  --no-pdf-header-footer --virtual-time-budget=4000 \
  --print-to-pdf=/tmp/boleta.pdf "file://$PWD/calculadora-boletas.html"
sips -s format png /tmp/boleta.pdf --out /tmp/boleta.png   # para revisarla como imagen
```

Para probar la lógica de cálculo de forma aislada conviene reimplementar las pocas
funciones puras (`tasaDeFecha`, redondeos) en un `node -e` en vez de montar un runner.

## Arquitectura

**`calcular()` es el único punto de recálculo** y alimenta **tres salidas** a partir de
los mismos números:

1. `#salida` — las filas en pantalla (se arman con el helper `fila()`).
2. `resumen` — la variable global de texto plano que usa `copiar()`.
3. `#hoja` — la hoja que solo se ve al imprimir (`@media print` oculta el resto).

Al agregar o cambiar una fila del resultado hay que tocar las tres; si no, la página, lo
copiado y lo impreso dejan de coincidir.

La fecha de la boleta manda sobre las dos cosas que dependen del tiempo: `sincronizarTasa()`
fija la tasa y `buscarUf()` trae el valor de la UF de ese día.

## Invariantes del dominio

No cambiar sin una razón explícita: son decisiones de cálculo, no detalles de estilo.

- **Pesos enteros al estilo SII**: el bruto se redondea primero y de ahí se derivan
  retención y líquido (`impuesto = round(bruto × t)`, `liquido = bruto − impuesto`), para
  que las tres filas calcen entre sí y con la boleta emitida. Redondear cada cifra por
  separado descuadra el desglose en $1.
- **Gross-up con `Math.ceil`**: partiendo del líquido deseado, el bruto se redondea hacia
  arriba para no quedar bajo el monto pedido.
- **Tasas de la Ley 21.133** (`TASAS`, 2019–2028): se derivan del año de la fecha y quedan
  editables. Fuera de ese tramo se usa la tasa del extremo más cercano, nunca 0%.
- **Fechas como texto `YYYY-MM-DD`**: `new Date(iso)` interpreta UTC y correría el día;
  usar `hoyISO()`, `anioDe()`, `aFormatoApi()` y `legible()`, que trabajan sobre el string.
- **UF desde mindicador.cl**: `GET /api/uf/DD-MM-YYYY`. `serie` vacía significa feriado o
  UF aún no publicada (más allá del día 9 del mes siguiente); en ese caso se cae al último
  valor vigente y se avisa. El valor de cada fecha se cachea en `localStorage` bajo
  `uf:YYYY-MM-DD`, así que una fecha ya consultada funciona sin conexión, y siempre se
  puede ingresar a mano.

## Convenciones

- Toda la interfaz y los comentarios van en español (formato de cifras `es-CL`: `$408.580`,
  `15,25%`, `10,00 UF`). Los helpers `clp()`, `uf()` y `pct()` son la única vía de formateo.
- La página no es un documento tributario: la hoja impresa lo dice explícitamente y esa
  aclaración debe sobrevivir a los cambios.
- No usar `alert`/`confirm`/`prompt`: el estado se comunica en los `<div class="status">`
  y en el texto de los botones.
