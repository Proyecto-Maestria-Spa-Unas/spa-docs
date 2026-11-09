# Matriz de trazabilidad de reglas de negocio
## Sistema de Inventario – Spa de Uñas

**Proyecto:** Sistema de Inventario – Spa de Uñas  
**Entregable:** D1 – Modelo entidad-relación final  
**Ubicación:** `docs/02-arquitectura/`  
**Versión:** 1.0  
**Fecha:** 9 de noviembre de 2026

---

## 1. Objetivo

Documentar la correspondencia entre las reglas de negocio RN-01 a RN-14 y los mecanismos implementados en la base de datos PostgreSQL de Supabase.

La matriz identifica las tablas, restricciones, claves foráneas, columnas calculadas, funciones, triggers y vistas que soportan cada regla.

## 2. Matriz de trazabilidad

| ID | Regla de negocio | Entidad o componente | Mecanismo de implementación | Migración / evidencia |
|---|---|---|---|---|
| RN-01 | Cada producto debe tener un identificador único. | `inventario.producto` | Clave primaria UUID e índice de unicidad sobre `codigo`. | `20261031120000_catalogo.sql` |
| RN-02 | Los campos definidos como obligatorios deben completarse antes de registrar un producto. | `inventario.producto` | Restricciones `NOT NULL` sobre código, nombre, categoría y unidad de medida; claves foráneas para categoría y unidad. | `20261031120000_catalogo.sql` |
| RN-03 | Todo movimiento debe estar asociado a un producto existente. | `inventario.movimiento` | Columna `producto_id NOT NULL` con clave foránea hacia `inventario.producto(id)`. | `20261107210000_movimientos.sql` |
| RN-04 | La cantidad de una entrada o salida debe ser mayor que cero. | `inventario.movimiento` | Restricción `CHECK (cantidad > 0)`. El trigger también valida que las cantidades sean enteras cuando la unidad de medida no admite decimales. | `20261107210000_movimientos.sql` |
| RN-05 | No se puede retirar una cantidad superior al stock disponible. | `inventario.movimiento` | Trigger `tg_movimiento_aplicar()`, que valida el stock antes de permitir una salida. | `20261107210000_movimientos.sql` |
| RN-06 | El stock actual no puede ser negativo. | `inventario.producto` y `inventario.movimiento` | Restricciones `CHECK` sobre stock y saldo resultante, junto con validaciones transaccionales del trigger de movimientos. | `20261031120000_catalogo.sql`; `20261107210000_movimientos.sql` |
| RN-07 | Cada movimiento debe registrar usuario, fecha y hora. | `inventario.movimiento` | `registrado_por NOT NULL` con FK hacia `auth.users(id)` y `ocurrido_en NOT NULL DEFAULT now()`. | `20261107210000_movimientos.sql` |
| RN-08 | Si el stock actual es mayor que el stock mínimo, el estado es Disponible. | `inventario.producto` | Columna generada `estado`, que evalúa las condiciones del stock y del producto. | `20261031120000_catalogo.sql` |
| RN-09 | Si el stock actual es menor o igual al stock mínimo y mayor que cero, el estado es Bajo stock. | `inventario.producto` | Expresión de la columna generada `estado`; la alerta de stock se consulta mediante `inventario.v_alerta_stock`. | `20261031120000_catalogo.sql`; `20261103090000_autorizacion_rls.sql` |
| RN-10 | Si el stock actual es cero, el estado es Agotado. | `inventario.producto` | Expresión de la columna generada `estado`, que evalúa `stock_actual <= 0` después de comprobar la descontinuación. | `20261031120000_catalogo.sql` |
| RN-11 | Descontinuado identifica un producto que ya no se maneja y debe diferenciarse de Agotado. | `inventario.producto` | Campo `descontinuado` y columna generada `estado`, que prioriza el estado DESCONTINUADO frente a AGOTADO. | `20261031120000_catalogo.sql` |
| RN-12 | La descontinuación debe conservar la información histórica del producto. | `inventario.producto`; `auditoria.registro_cambio` | La descontinuación se registra mediante el campo `descontinuado`, sin eliminar el producto. El trigger de auditoría registra operaciones INSERT, UPDATE y DELETE sobre producto. | `20261031120000_catalogo.sql`; `20261108090000_auditoria_cambios.sql` |
| RN-13 | La eliminación de movimientos, si se habilita, debe estar restringida al rol autorizado. | `inventario.movimiento`; `seguridad` | La implementación actual no habilita la eliminación: un trigger impide UPDATE, DELETE y TRUNCATE. Las anulaciones se registran como nuevos movimientos. | `20261107210000_movimientos.sql` |
| RN-14 | El nivel mínimo debe estar definido para que pueda generarse una alerta de bajo stock. | `inventario.producto`; `inventario.v_alerta_stock` | `stock_minimo` es nullable. La vista de alertas considera productos con stock mínimo definido, stock actual menor o igual al mínimo y no descontinuados. | `20261031120000_catalogo.sql`; `20261103090000_autorizacion_rls.sql` |

## 3. Observaciones técnicas

### 3.1. Reglas implementadas mediante restricciones

Las siguientes reglas tienen soporte mediante restricciones declarativas de la base de datos:

- RN-01: unicidad del código del producto y clave primaria.
- RN-02: obligatoriedad de campos y relaciones de categoría y unidad de medida.
- RN-03: asociación obligatoria del movimiento con un producto existente.
- RN-04: cantidad estrictamente positiva.
- RN-06: stock y saldo resultante no negativos.

### 3.2. Reglas implementadas mediante lógica transaccional

Las reglas que requieren validar el estado del inventario durante una operación se implementan mediante el trigger `tg_movimiento_aplicar()`:

- RN-05: disponibilidad suficiente para realizar salidas.
- RN-06: actualización transaccional del stock.
- RN-07: registro de usuario y fecha del movimiento.
- RN-04: validación de cantidades enteras cuando la unidad no admite decimales.

El trigger también aplica el cálculo del costo promedio ponderado para las entradas de compra y conserva el resultado de la valoración en cada movimiento.

### 3.3. Reglas implementadas mediante columnas calculadas y vistas

- RN-08, RN-09, RN-10 y RN-11: se soportan mediante la columna generada `estado` de `inventario.producto`.
- RN-14: se soporta mediante la vista `inventario.v_alerta_stock`, que identifica productos que cumplen las condiciones de alerta.

### 3.4. Conservación histórica y eliminación de movimientos

Para RN-12, la auditoría registra cambios y eliminaciones de productos. La descontinuación se maneja mediante un atributo, por lo que no requiere eliminar el registro del producto.

Para RN-13, los movimientos son inmutables. La base de datos impide su modificación y eliminación física, y las anulaciones se registran mediante nuevos movimientos. Por tanto, no existe actualmente un procedimiento de eliminación física sujeto a autorización por rol.

## 4. Componentes relacionados con el modelo entidad-relación

El modelo incluye las siguientes entidades físicas:

**Esquema `inventario`:**
- categoria
- unidad_medida
- producto
- proveedor
- motivo_movimiento
- movimiento

**Esquema `seguridad`:**
- rol
- permiso
- rol_permiso
- perfil

**Esquema `auditoria`:**
- registro_cambio

**Entidad externa de Supabase:**
- auth.users

Las vistas `v_inventario`, `v_alerta_stock`, `v_kardex` y `v_consumo_diario` son componentes de consulta y no tablas físicas.

## 5. Conclusión

La matriz establece la trazabilidad entre las catorce reglas de negocio y los mecanismos implementados en la base de datos. Se identifican restricciones declarativas, claves foráneas, columnas generadas, funciones, triggers y vistas como mecanismos de aplicación y consulta de las reglas.

La documentación distingue entre las reglas que se garantizan mediante restricciones y aquellas cuya aplicación depende de lógica transaccional o componentes de consulta. Esta trazabilidad complementa el modelo entidad-relación final y facilita su revisión técnica y funcional.
