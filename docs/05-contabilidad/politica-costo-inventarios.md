# A3 · Política de valoración del inventario: costo promedio ponderado

| | |
|---|---|
| **Tarea** | A3 · spa-docs#31 |
| **Responsable** | ivonnelizethmoreno-cloud (contadora) |
| **Decisión de origen** | DEC-11, ratificada en el acta A1 (`docs/02-arquitectura/acta-A1-decisiones.md`) |
| **Alcance** | Solo la valoración del inventario (costo de cada movimiento y saldo tipo kardex). No incluye libros contables, facturación ni compras como proceso financiero |
| **Marco** | Microempresa (Grupo 3): anexo 3 del Decreto 2420 de 2015, capítulo 8 |
| **Estado** | Criterios 1 y 2 cumplidos. Criterio 3 («validada con D7 y X7») pendiente hasta que existan D7 y X7 |

## 1. Fundamento normativo

### 1.1 Grupo del cliente: microempresa (Grupo 3)

- El cliente es una **microempresa**, según lo indicado por el equipo el 03-nov-2026. El acta A1 lo registra en DEC-11.
- **Condición que debe cumplir el cliente.** El Decreto 1670 de 2021 modificó el artículo 1.1.3.1 del Decreto 2420 de 2015. Desde el 1-ene-2023, el Grupo 3 se define por los ingresos según el sector, más condiciones como no tener subsidiarias ni estar obligado a consolidar. Para el **sector servicios**, el Decreto 957 de 2019 (artículo 2.2.1.13.2.2 del Decreto 1074 de 2015) fija como microempresa la que tiene ingresos por actividades ordinarias **iguales o inferiores a 32.988 UVT** al año.
- **Marco aplicable:** el *Marco técnico normativo de información financiera para las microempresas*. Es el anexo 3 del Decreto 2420 de 2015 y proviene del Decreto 2706 de 2012.

### 1.2 Capítulo 8 · Inventarios (anexo 3)

| Numeral | Qué dispone | Cómo lo aplica esta política |
|---|---|---|
| 8.1 | Son inventarios los materiales o suministros que se consumen en la prestación de servicios | Los insumos del spa (esmaltes, acetona, limas, etc.) son inventario |
| 8.2 | Los inventarios se miden al costo | El sistema registra el costo de cada movimiento |
| 8.5 | El costo de adquisición incluye el precio de compra, los impuestos no recuperables, el transporte, la manipulación y otros costos directamente atribuibles. Los descuentos comerciales y las rebajas se restan | El `costo_unitario` de cada COMPRA debe ser ese costo neto (sección 5, punto 4) |
| 8.7 | Se puede usar el sistema periódico o el permanente. Con el periódico hay que hacer al menos una toma física anual | El sistema es **permanente**: cada movimiento actualiza el saldo. Los conteos físicos se registran como `AJUSTE_CONTEO` |
| 8.8 | Se puede usar PEPS, costo promedio ponderado u otro método de reconocido valor técnico, aplicado de forma uniforme a todos los inventarios. **UEPS no está permitido** | Se usa **costo promedio ponderado** para todos los productos (DEC-11) |
| 8.9 | Al cierre de cada periodo se evalúa el deterioro (daño, obsolescencia, precios a la baja), medido según los numerales 2.34 a 2.36 | El deterioro contable queda fuera del alcance del sistema. Las bajas físicas se registran como `MERMA` o `VENCIMIENTO` |
| 8.10 | El valor en libros se reconoce como costo en el periodo del ingreso correspondiente | Cada salida se valora al promedio vigente |

### 1.3 Relación con la NIIF para las Pymes, Sección 13

El criterio 1 de A3 cita la NIIF para las Pymes, Sección 13. Ese es el marco del Grupo 2 (anexo 2). Para una microempresa, la norma principal es el capítulo 8 del anexo 3. El numeral 3.1 del anexo 3 remite a la NIIF para las Pymes solo para transacciones que el marco de microempresas no trata.

En cuanto a la fórmula de costo, los dos marcos coinciden: ambos admiten PEPS o promedio ponderado y ninguno admite UEPS (párrafos 13.17 y 13.18). Por eso, la decisión de DEC-11 es válida en los dos.

### 1.4 Fuentes

- Anexo 3 del Decreto 2420 de 2015, capítulos 1, 3 y 8. Se consultó una copia digitalizada en [Actualícese](https://cdn.actualicese.com/normatividad/2015/Decretos/D2420-15/Anexo-3-D2420-15.pdf). Antes de la entrega al cliente conviene cotejarla con el Diario Oficial.
- [Decreto 1670 de 2021](https://cancilleria.gov.co/sites/default/files/Normograma/docs/decreto_1670_2021.htm): requisitos del Grupo 3.
- [Decreto 957 de 2019](https://normograma.invima.gov.co/compilacion/docs/decreto_0957_2019.htm): rangos de ingresos por sector.
- [Gerencie · Sección 13 NIIF para pymes](https://www.gerencie.com/que-debemos-saber-de-inventarios-seccion-13-niif-para-pymes.html): párrafos 13.17 a 13.19 (fuente secundaria).

## 2. Fórmula elegida y por qué

El sistema usa **costo promedio ponderado móvil**: el promedio se recalcula con cada entrada que trae costo. Es una de las fórmulas que admite el numeral 8.8 del anexo 3 y es la que ya implementan el esquema y los contratos:

- La columna `inventario.producto.costo_promedio` (precisión de 4 decimales) existe en la base.
- El contrato `analitica-v1` define el kardex con `metodo = PROMEDIO_PONDERADO` y la fórmula `(saldo anterior × promedio + entrada) ÷ saldo`.
- El contrato `movimientos-v1` exige `costo_unitario` en las entradas por `COMPRA`, porque alimentan el promedio.

Se aplica la misma fórmula a todos los productos, como exige el numeral 8.8 («de forma uniforme a todos los inventarios»).

## 3. Reglas de cálculo

| Movimiento | Efecto en el saldo | Costo con que se registra | Efecto en el costo promedio |
|---|---|---|---|
| Entrada `COMPRA` | Suma la cantidad | `costo_unitario` de la compra (obligatorio) | Se recalcula: `(saldo anterior × promedio anterior + cantidad × costo unitario) ÷ saldo nuevo` |
| Salida (`CONSUMO_SERVICIO`, `MERMA`, `VENCIMIENTO`) | Resta la cantidad | Promedio vigente | No cambia |
| Valor del saldo | — | — | `saldo × costo_promedio` |

- **Redondeo:** el costo promedio se guarda con 4 decimales y las cantidades con 3 (contratos y migración del catálogo). Por eso el valor del saldo puede diferir en centésimas del valor exacto (ver el ejemplo).
- Si el saldo llega a 0, la siguiente compra fija el promedio en su propio costo unitario, porque el saldo anterior no aporta valor.

## 4. Ejemplo numérico del kardex

Producto: acetona (unidad: ml), con un saldo inicial de 350 ml a 38,0000. Los valores coinciden con los ejemplos del contrato `analitica-v1` (compra de 150 a 42,0000 → saldo 500 → promedio 39,2000 → valor 19.600,0000).

| # | Movimiento | Cantidad | Costo unitario | Costo total | Saldo | Costo promedio | Valor del saldo |
|---|---|---|---|---|---|---|---|
| 0 | Saldo existente | — | — | — | 350,000 | 38,0000 | 13.300,0000 |
| 1 | Entrada · COMPRA | 150,000 | 42,0000 | 6.300,0000 | 500,000 | **39,2000** | 19.600,0000 |
| 2 | Salida · CONSUMO_SERVICIO | 15,000 | 39,2000 | 588,0000 | 485,000 | 39,2000 | 19.012,0000 |
| 3 | Entrada · COMPRA | 100,000 | 45,0000 | 4.500,0000 | 585,000 | **40,1915** | 23.512,0275 |
| 4 | Salida · MERMA | 85,000 | 40,1915 | 3.416,2775 | 500,000 | 40,1915 | 20.095,7500 |

Cálculos de las filas que cambian el promedio:

- Fila 1: (350 × 38,0000 + 6.300,0000) ÷ 500 = 19.600,0000 ÷ 500 = **39,2000**
- Fila 3: (485 × 39,2000 + 4.500,0000) ÷ 585 = 23.512,0000 ÷ 585 = 40,191452… → **40,1915** (4 decimales). Por ese redondeo, el valor del saldo es 23.512,0275 y no 23.512,0000.

## 5. Puntos abiertos para D7 y X7

Estos casos no están definidos en los contratos ni en el plan. La política propone una regla para cada uno; se confirma al validar con D7 y X7 (criterio 3).

| # | Caso | Lo verificado | Regla propuesta |
|---|---|---|---|
| 1 | **Saldo inicial al crear el producto** | `POST /productos` acepta `cantidad_inicial` pero **no tiene campo de costo** (contrato `catalogo-v1`). Sin costo, ese saldo entraría con promedio 0 | El saldo inicial debe registrarse con su costo. Requiere decidir en B9 y D7 cómo recibirlo |
| 2 | **Entrada `AJUSTE_CONTEO` o `SALDO_INICIAL` sin costo** | En `movimientos-v1`, `costo_unitario` solo es obligatorio en `COMPRA` | Se valora al promedio vigente; el promedio no cambia |
| 3 | **Anulación de una COMPRA** | La anulación revierte el saldo (HU-13, D10), pero no está definido su efecto en el promedio | El movimiento compensatorio usa el mismo costo unitario de la compra anulada y el promedio se recalcula con la misma fórmula. Si después hubo salidas, D7 debe definir el recálculo |
| 4 | **Componentes del costo de la COMPRA** (resuelto) | El numeral 8.5 incluye en el costo los impuestos no recuperables y el transporte, y resta los descuentos comerciales. El contrato `movimientos-v1` solo recibe un `costo_unitario` | Quien registra la compra digita el costo unitario **neto**: precio menos descuentos comerciales, más fletes y **más el IVA pagado**. El cliente **no es responsable de IVA** (confirmado por Ivonne Moreno, contadora, el 03-nov-2026), así que el IVA de las compras no es descontable y forma parte del costo |

## 6. Validación (criterio 3)

- [ ] **D7:** el trigger de movimientos recalcula `costo_promedio` y guarda `costo_promedio_resultante` según la sección 3.
- [ ] **D10:** la prueba SQL «Costo promedio correcto con el ejemplo de A3» reproduce la tabla de la sección 4.
- [ ] **X7:** `GET /analitica/kardex` devuelve las mismas líneas y el mismo valor final que la sección 4.
