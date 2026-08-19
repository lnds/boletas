# Calculadora de boletas de honorarios

Calculadora local para boletas de honorarios en Chile. Es un único archivo HTML
sin dependencias ni servidor: se abre directamente en el navegador.

```
open calculadora-boletas.html
```

## Qué calcula

A partir de un monto en UF y una fecha, entrega el bruto de la boleta, el
impuesto y el líquido, en los dos escenarios posibles:

- **El cliente retiene** (empresa o institución): entera la retención en el SII
  y deposita el líquido.
- **El cliente no retiene** (persona natural): deposita el bruto y el PPM se
  declara en el Formulario 29 del mes siguiente.

El monto en UF se puede ingresar como bruto o como líquido deseado; en el
segundo caso se hace el *gross-up* (`bruto = líquido / (1 - tasa)`).

Con el botón **Imprimir** se genera una hoja limpia con el detalle del
cálculo: los datos de entrada, el desglose y el total, sin los controles de la
página. Es un respaldo del cálculo, no un documento tributario.

## Criterios de cálculo

- **Tasa de retención**: la vigente según la Ley 21.133, que la sube de forma
  gradual hasta 17% en 2028. Se deriva del año de la fecha de la boleta y queda
  editable. Fuera del tramo 2019–2028 se usa la tasa del extremo más cercano.
- **Pesos enteros**: el bruto se redondea primero y de ahí se derivan retención
  y líquido, tal como opera el SII, para que las cifras calcen con la boleta
  emitida. Partiendo del líquido deseado el bruto se redondea hacia arriba, para
  no quedar bajo el monto pedido.
- **Valor de la UF**: se consulta a [mindicador.cl](https://mindicador.cl) para
  la fecha elegida y se guarda en `localStorage`, de modo que una fecha ya
  consultada funciona sin conexión. Siempre se puede ingresar a mano.

Cálculo referencial: la retención efectiva es la vigente a la fecha de emisión
en el SII.
