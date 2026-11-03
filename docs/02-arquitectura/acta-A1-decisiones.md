# Acta A1 · Ratificación de decisiones DEC-01 a DEC-12

| | |
|---|---|
| **Tarea** | A1 · spa-docs#28 |
| **Responsable** | ivonnelizethmoreno-cloud |
| **Fecha y hora** | 02-nov-2026, 11:55 p. m. |
| **Asistentes** | Jhon Soler, Ivonne Moreno |
| **Redactó el acta** | Ivonne Moreno |
| **Aprobación** | Publicada mediante PR aprobado por QA (ADR-0007) |

El **valor vigente** de cada decisión es lo que está en el código, los contratos, las migraciones o el documento de arquitectura, verificado el 01 y el 02 de noviembre de 2026. Según el criterio de A1, las decisiones no discutidas adoptan el valor del documento de arquitectura.

## Decisiones

| DEC | Pregunta (issue) | Valor vigente y fuente | Decisión del grupo |
|---|---|---|---|
| DEC-01 (#14) | ¿El rol Gerente forma parte definitiva del sistema? | Sí existe: `CodigoRol.GERENTE` en el backend, y la migración de D5 lo siembra con 5 permisos (`usuario.gestionar`, `producto.consultar`, `movimiento.consultar`, `alerta.consultar`, `analitica.consultar`) | Se ratifica |
| DEC-02 (#15) | ¿Qué rol asume la gestión de usuarios si se elimina Gerente? | El Administrador ya tiene `usuario.gestionar` (sección 6 del documento de arquitectura y migración de D5). Solo aplica si DEC-01 cambia | Se ratifica |
| DEC-03 (#16) | ¿La Manicurista puede registrar entradas o solo salidas y consumos? | Solo salidas: la Manicurista no tiene `movimiento.registrar_entrada` (sección 6 y migración de D5) | Se ratifica |
| DEC-04 (#17) | ¿Las alertas de bajo stock son obligatorias u opcionales? | Opcionales por producto: `stock_minimo` nulo = sin alerta (RN-14, contrato y migración del catálogo). El módulo de alertas está en el alcance (X4) y la Manicurista tiene `alerta.consultar` | Se ratifica |
| DEC-05 (#18) | ¿El filtro de movimientos por fechas es obligatorio? | Opcional: `GET /movimientos` tiene los filtros `desde` y `hasta`, no obligatorios (contrato `movimientos-v1`) | Se ratifica |
| DEC-06 (#19) | ¿Eliminar movimientos o anulación o corrección controlada? | Anulación controlada: no existe `DELETE /movimientos` (responde 405) y existe `POST /movimientos/{id}/anulaciones` | Se ratifica. ADR-0003 pasa a «Aceptado» en este mismo PR |
| DEC-07 (#20) | Valores medibles de rendimiento y disponibilidad | No había valores definidos. Línea base medida el 03-nov-2026 (1 usuario, 50 solicitudes a `/api/v1/health/ready` en staging): p50 = 601 ms, p95 = 763 ms, 0 % de errores | p95 máximo de 1.500 ms y errores máximos de 1 %, medidos con 20 usuarios simultáneos en Q5. Disponibilidad: no se mide en este proyecto |
| DEC-08 (#21) | Políticas de contraseñas, recuperación de acceso y sesiones | Contraseña de 8 a 128 caracteres (contrato `identidad-v1`); sesión con JWT de 30 minutos (`JWT_EXPIRES_MINUTES=30`); recuperación por correo con Supabase Auth (`POST /auth/recuperar`, X3) | Se ratifica |
| DEC-09 (#22) | Estrategia técnica de respaldo y recuperación | Respaldo semanal cifrado (AES-256) de staging, guardado 30 días (`spa-database/.github/workflows/respaldo.yml`). Todavía no se ejecuta: está solo en `develop`, y GitHub ejecuta los workflows programados desde `main` | Se ratifica. Respaldo semanal cifrado; empieza a ejecutarse con la primera release a `main`. Producción: el mismo esquema dentro de DV3 |
| DEC-10 (#23) | Catálogo definitivo de categorías y productos | 8 categorías y 12 unidades de medida (migración `20261031130000_catalogo_datos.sql`, en staging). Los productos de demostración son la tarea D11 | Se ratifica |
| DEC-11 (#29) | ¿Incluir la valoración del inventario (costo promedio ponderado) en el alcance? | El issue propone incluir solo la valoración (costo de cada movimiento y saldo tipo kardex), sin libros contables ni facturación. La NIIF para las Pymes, Sección 13 (párr. 13.17 y 13.18), admite PEPS o costo promedio ponderado y no permite UEPS ([Gerencie](https://www.gerencie.com/que-debemos-saber-de-inventarios-seccion-13-niif-para-pymes.html)). La columna `inventario.producto.costo_promedio` ya existe | Costo promedio ponderado, solo como valoración del inventario. Propuesto y validado por Ivonne Moreno, contadora, el 02-nov-2026. Grupo del cliente: microempresa (Grupo 3), precisado el 03-nov-2026. Su marco es el anexo 3 del Decreto 2420 de 2015, cuyo numeral 8.8 también admite el costo promedio ponderado y no permite UEPS. Detalle en la política A3 (`docs/05-contabilidad/politica-costo-inventarios.md`) |
| DEC-12 (sin issue) | ¿Qué decide DEC-12? | El criterio de A1 decía «DEC-12 = proveedores georreferenciados»; la revisión 16 del documento de arquitectura la usaba para la extensión a varias sedes | DEC-12 = proveedores georreferenciados (D12, X2, X6). La extensión a varias sedes queda fuera del alcance. No se crea issue: esta acta es su registro |

## Comunicación

Criterio de A1: la decisión se comunica el mismo día a backend, frontend y datos.

| Equipo | Integrantes | Comunicado por | Fecha |
|---|---|---|---|
| Backend | Damx0611, ivonnelizethmoreno-cloud | Ivonne Moreno | 3/11/2026 |
| Frontend | Gleidy725, Estefa44445 | Ivonne Moreno | 3/11/2026 |
| Datos | mikeforeros | Ivonne Moreno | 3/11/2026 |

## Cierre

- Al fusionarse el PR de esta acta se cierran A1 (#28) y los issues DEC-01 a DEC-11 (#14 a #23 y #29), enlazados con `Closes`.
- Este mismo PR pasa ADR-0003 a «Aceptado» y actualiza la sección 2 del documento de arquitectura con el significado de DEC-12.
