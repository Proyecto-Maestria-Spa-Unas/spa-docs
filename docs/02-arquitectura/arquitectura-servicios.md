# Arquitectura de servicios y contratos de API · Spa de Uñas

Revisión 16 · 01-nov-2026 · @Jhon Jairo Soler

> **Cómo leer este documento.** Describe el diseño acordado y el estado verificado el 01-nov-2026 contra los repositorios. Cuando algo está pendiente se dice con su tarea (por ejemplo «pendiente: D5»). Si hay contradicción, mandan el código, los contratos (`spa-backend-api/docs/openapi/`) y las migraciones (`spa-database/supabase/migrations/`).
>
> Cambios frente a la revisión 15 (25-oct): plan de 2 entregas en lugar de 4 sprints; 37 endpoints con su permiso; reparto de endpoints por tarea; estado real de respaldos, keep-alive, seguimiento, Kubernetes y producción; cliente HTTP con axios; regla `Refs`/`Closes` para issues.

## 1. Decisión de arquitectura

El backend se divide en **4 servicios de dominio** (Identidad, Catálogo, Movimientos y Alertas y analítica) más **S0 · Datos**, el servicio de plataforma que recibe la base de datos. Los 4 servicios de dominio se despliegan juntos como **monolito modular** en un solo proyecto de Vercel (`spa-unas-api`), mientras S0 se despliega aparte con su propio pipeline. En la Entrega 2 esa misma imagen de la API se ejecuta en Kubernetes con Istio (DV5, clúster local) y en una demostración en AWS EKS (DV7). Separar cada servicio en su propio microservicio (`spa-svc-<dominio>`) **no está en el plan de entregas**; el diseño lo permite sin cambiar las rutas que consume el frontend (sección 10).

Por qué así y no con microservicios desde el primer día:

- **Equipo pequeño.** Hay 2 desarrolladores de backend. Cuatro repositorios, cuatro pipelines y cuatro proyectos de Vercel multiplicarían el trabajo de operación sin agregar funcionalidad.
- **Consistencia.** Una salida de inventario debe bloquear el producto y descontar el stock en la **misma transacción** (RN-05, RN-06). Ese invariante lo garantiza la base con un trigger, no la red entre servicios.
- **Extracción sin reescritura.** Cada módulo ya tiene sus propias capas hexagonales, su contrato OpenAPI, su prefijo de rutas y sus tablas. Extraerlo sería copiar la carpeta a un repositorio creado desde `spa-template-servicio` (el repositorio existe) y agregar una ruta en Istio.
- **Reglas verificables.** Un módulo **nunca** importa el código interno de otro. Lo comprueba `import-linter` en el check `calidad / pipeline`, así que la frontera se puede auditar desde el primer commit.

Esta decisión se cita como **ADR-0009 · Monolito modular con 4 servicios de dominio**. El ADR aún no está redactado en `spa-docs/docs/02-arquitectura/adr/` (allí están del 0000 al 0006); mientras tanto, esta sección es su registro.

## 2. Mapa de servicios

Cada servicio es dueño de un conjunto de funcionalidades, de sus rutas y de sus tablas. La traza de cada servicio sale de las descripciones de sus endpoints en los contratos. Algunos requerimientos se comparten entre servicios (por ejemplo RF-14, RF-17 y RF-19), porque los cubren endpoints de servicios distintos.

| Servicio | Responsabilidad | Historias | Requerimientos y reglas (según contratos) | Prefijos de ruta | Entregas y tareas |
| --- | --- | --- | --- | --- | --- |
| **S0 · Datos** | Recibe la base de datos: esquema, migraciones, datos de referencia, roles y permisos de base, vistas de consulta, respaldos y despliegue | Todas (soporte) | Reglas de negocio que se garantizan en la base | Sin rutas HTTP: su contrato es el esquema versionado | Entrega 1: D0 a D6 · Entrega 2: D7 a D13 |
| **S1 · Identidad** (9 endpoints) | Autenticación, emisión de JWT, recuperación de contraseña, usuarios, roles y permisos | HU-01, HU-09 | RF-01, RF-13, RNF-07 | `/auth`, `/usuarios`, `/roles` | Entrega 1: B5, B6 · Entrega 2: X3 |
| **S2 · Catálogo** (14 endpoints) | Productos, categorías, unidades, stock mínimo, descontinuación, estado del inventario y proveedores | HU-02 a HU-05, HU-12 | RF-02 a RF-05, RF-11, RF-12, RF-16, RF-17, RF-19; RN-01, RN-02, RN-11, RN-12, RN-14; RNF-09 | `/productos`, `/categorias`, `/unidades-medida`, `/proveedores` | Entrega 1: B4, B8, B9 · Entrega 2: X2 |
| **S3 · Movimientos** (6 endpoints) | Entradas, salidas, validación de stock, historial y anulación controlada | HU-06 a HU-08, HU-11, HU-13 | RF-06 a RF-10, RF-14, RF-15, RF-20; RN-13 | `/movimientos`, `/motivos-movimiento` | Entrega 2: B10, B11, X1 |
| **S4 · Alertas y analítica** (8 endpoints) | Alertas de bajo stock y agotados, tablero, consumo, cobertura, kardex valorizado, proveedores cercanos y exportación | HU-10, HU-11 | RF-14, RF-17, RF-18, RF-19; RNF-10; DEC-11 | `/alertas`, `/analitica` | Entrega 2: B14, X4, X5, X6, X7 |

Todas las rutas HTTP cuelgan de `/api/v1`. La base de datos la recibe **S0 · Datos** (`spa-database`, a cargo de mikeforeros): crea y versiona el esquema y lo despliega con su propio pipeline. S1 a S4 se conectan solo a sus propias tablas (sección 3).

La **georreferenciación de proveedores** sí está en la Entrega 2 (D12, X2, X6): tabla `inventario.proveedor` con ubicación PostGIS y búsqueda de proveedores cercanos. La extensión a **varias sedes o bodegas** no está en el plan: la especificación describe un solo establecimiento, así que requiere una decisión nueva (DEC-12). PostGIS ya está instalado en la migración base.

## 3. Propiedad de los datos y acceso a la base

La base de datos la recibe y administra **S0 · Datos**, el repositorio `spa-database`. Mike escribe las migraciones y el pipeline `cd` las aplica en Supabase. Los servicios S1 a S4 nunca ejecutan DDL: solo leen y escriben las tablas que les pertenecen.

### S0 · Servicio de Datos

S0 es un **servicio de plataforma**. Tiene repositorio, responsable, pruebas y pipeline de despliegue propios, y se publica en su propio ciclo. Su contrato con los demás servicios es el **esquema versionado** (tablas, vistas y roles), no una API HTTP.

| Componente | Qué entrega | Dónde vive | Responsable |
| --- | --- | --- | --- |
| Migraciones | Tablas, restricciones, triggers y **datos de referencia** (categorías, unidades, motivos, roles y permisos) con `INSERT ... ON CONFLICT DO NOTHING` | `supabase/migrations/` | mikeforeros |
| Seed | Solo datos de demostración. Supabase ejecuta `seed.sql` una sola vez por proyecto; si el archivo cambia después, no se vuelve a ejecutar | `supabase/seed.sql` | mikeforeros |
| Pruebas de base | Pruebas SQL en CI sobre un PostGIS limpio, antes de cada fusión | `tests/sql/`, `ci.yml` | mikeforeros |
| Despliegue | `develop` → staging automático; `main` → producción con aprobación de QA | `.github/workflows/cd.yml` | Jhon (pipeline), mikeforeros (contenido) |
| Roles y permisos | RLS en las tablas (activa en el catálogo); un rol por servicio con `GRANT` mínimo (pendiente: D13) | Migraciones | mikeforeros; Jhon custodia las contraseñas |
| Vistas de consulta | `v_inventario` y `v_alerta_stock`, el contrato de lectura para S2 y S4 (pendiente: D6) | Migraciones | mikeforeros |
| Respaldos | Volcado semanal de staging (domingos 06:41, hora de Bogotá), cifrado con AES-256 y guardado 30 días como artefacto | `respaldo.yml`, hoy solo en `develop` | Jhon |
| Disponibilidad | Consulta diaria a staging (08:17, hora de Bogotá) para que el plan Free de Supabase no pause el proyecto | `keep-alive.yml`, hoy solo en `develop` | Jhon |
| Detección de deriva | Comparar el esquema de staging con las migraciones (`supabase db diff`) | No existe y no está en el plan de entregas | — |

**Importante:** GitHub solo ejecuta los workflows programados desde la rama por defecto (`main`), y en `spa-database` la rama `main` hoy solo tiene `ci.yml`. Por eso `respaldo.yml` y `keep-alive.yml` **todavía no se ejecutan**: empiezan a correr cuando la primera release lleve `develop` a `main`.

Por qué S0 no expone un CRUD por HTTP para que los otros servicios lean y escriban a través de él:

- **Atomicidad.** Registrar una salida y descontar el stock debe ocurrir en una sola transacción. Con un servicio de datos intermedio serían dos llamadas de red, y una podría fallar después de la otra.
- **Latencia.** En Vercel cada llamada entre funciones suma un arranque en frío. Cada operación del inventario pagaría el viaje dos veces.
- **Acoplamiento.** Si ese servicio cae, caen los cuatro dominios. Sería un punto único de falla, justo lo que los microservicios buscan evitar.
- **Duplicación.** Las reglas de negocio quedarían repartidas entre el dominio y el servicio de datos.

Si en el futuro alguien necesita leer datos desde afuera (por ejemplo QGIS o ArcGIS para la extensión geográfica, o un tablero externo), S0 lo publica como **vistas de solo lectura**, o mediante la Data API de Supabase con RLS. Nunca como un CRUD de escritura.

| Tabla o vista (esquema) | Dueño que escribe | Puede leer | Regla |
| --- | --- | --- | --- |
| `auth.users` (Supabase Auth) | S1, por la API de Supabase Auth | S1 | Nadie la toca por SQL |
| `seguridad.rol`, `permiso`, `rol_permiso` | Migraciones de `spa-database` (pendiente: D5) | S1 | Los permisos cambian por migración, no por la API |
| `seguridad.perfil` | S1 (pendiente: D5) | S1; los demás, solo el nombre del responsable | Desactivar en lugar de borrar |
| `inventario.categoria`, `unidad_medida` | S2 | Todos | Existen en staging. Datos de referencia en la migración `20261031130000_catalogo_datos.sql`: 8 categorías y 12 unidades |
| `inventario.producto` | S2, excepto `stock_actual`, `costo_promedio`, `estado` y `version` | Todos | Existe en staging. `estado` es columna generada (RN-08 a RN-11) y `version` sube con cada cambio. `stock_actual` y `costo_promedio` solo los cambia el trigger de movimientos (pendiente: D7) |
| `inventario.proveedor` | S2 (pendiente: D12) | S2, S4 | NIT único y ubicación `geography(Point, 4326)` con índice GIST |
| `inventario.proveedores_cercanos()` | Nadie (función) | S4 (pendiente: D12) | Búsqueda por radio con `ST_DWithin` |
| `inventario.motivo_movimiento` | Migraciones de `spa-database` (pendiente: D12) | S3 | Datos de referencia por migración, no por seed |
| `inventario.movimiento` | S3, solo `INSERT` (pendiente: D7) | S3, S4 | Libro mayor inmutable: se corrige con `ANULACION`, nunca con `UPDATE` ni `DELETE` |
| `inventario.v_inventario`, `v_alerta_stock` | Nadie (vistas, pendiente: D6) | S2, S4 | `security_invoker = true` |
| `auditoria.registro_cambio` | Solo triggers (pendiente: D8) | S4, con permiso de auditoría | Solo inserción |

**Conexión por fase:**

- **Entregas 1 y 2 (Vercel).** Una sola variable `DB_URL` hacia el pooler de transacciones de Supabase (puerto 6543), con `prepare_threshold=None` en psycopg. La propiedad de las tablas se garantiza con revisión de código y con `import-linter`: cada módulo tiene su propio repositorio SQLAlchemy y no comparte modelos con otro módulo.
- **Roles por servicio (D13, Entrega 2).** Un rol de base por servicio (`svc_identidad`, `svc_catalogo`, `svc_movimientos`, `svc_analitica`) con `GRANT` solo sobre sus tablas, y `svc_analitica` en solo lectura, con pruebas SQL de que ninguno accede a lo ajeno. Mike los crea por migración y Jhon custodia sus contraseñas como secretos. El plan no define que la API cambie a esos roles.
- **Consistencia entre servicios (D7).** Cuando S3 registra una salida, el trigger `trg_movimiento_aplicar` bloquea el producto (`FOR UPDATE`), valida el stock y lo actualiza en la misma transacción. S2 nunca recalcula el stock: lo lee.

## 4. Convenciones del contrato API

El contrato se escribe primero (contract-first) y es la única fuente de verdad entre backend y frontend. Estas convenciones son obligatorias en los 4 servicios.

| Tema | Regla |
| --- | --- |
| Base y versión | `/api/v1/...`. Un cambio incompatible crea `/api/v2`; la versión anterior se mantiene al menos un sprint |
| Recursos | Sustantivos en plural y kebab-case (`/unidades-medida`). Las acciones de negocio son subrecursos con `POST` (`/productos/{id}/descontinuar`) |
| JSON | Campos en `snake_case`, igual que la base. Las cantidades y los costos viajan como **texto decimal** (`"12.500"`) para no perder precisión. Las fechas en ISO 8601 con zona (`2026-10-25T18:30:00-05:00`) |
| Paginación | `?page=1&size=20` (máximo 100). Respuesta `{ "items": [], "total": 0, "page": 1, "size": 20 }` |
| Filtros | Por query string (`?categoria_id=3&estado=BAJO_STOCK`). Cada endpoint declara sus filtros en el contrato; la versión 1 no tiene parámetro de orden |
| Errores | `application/problem+json`: `{ type, title, status, detail, code, errors[] }`. El campo `code` es estable y es el que usa el frontend |
| Autenticación | `Authorization: Bearer <jwt>` en todo excepto `/auth/login`, `/auth/recuperar`, `/health`, `/health/ready`, `/docs` y `/openapi.json`. JWT HS256 de 30 minutos con `sub`, `rol` y `permisos` |
| Autorización | Cada endpoint declara su permiso con `Depends(requiere("producto.crear"))`. Sin permiso responde `403` aunque se llame directamente (RNF-07) |
| Idempotencia | Los `POST` de movimientos exigen la cabecera `Idempotency-Key` (UUID). Un reintento con la misma clave devuelve el mismo movimiento |
| Concurrencia | `PATCH /productos/{id}` exige `If-Match: <version>`. Si la versión no coincide responde `412` |
| Documentación viva | `/api/v1/docs` (Swagger) y `/api/v1/openapi.json`, generados por FastAPI |

**Códigos de error del dominio:**

| `code` | HTTP | Cuándo |
| --- | --- | --- |
| `CREDENCIALES_INVALIDAS` | 401 | Usuario o contraseña incorrectos (HU-01) |
| `NO_AUTENTICADO` | 401 | Falta el token, está vencido o es inválido |
| `SIN_PERMISO` | 403 | El rol no tiene el permiso del endpoint (RF-01) |
| `USUARIO_INACTIVO` | 403 | La cuenta está desactivada (HU-09) |
| `RECURSO_NO_EXISTE` | 404 | Producto, usuario o movimiento inexistente (RN-03) |
| `CODIGO_DUPLICADO` | 409 | Código de producto o correo ya registrado (RN-01) |
| `STOCK_INSUFICIENTE` | 409 | Salida mayor al stock disponible (RN-05, RF-08) |
| `PRODUCTO_DESCONTINUADO` | 409 | Movimiento sobre un producto descontinuado (HU-12) |
| `MOVIMIENTO_YA_ANULADO` | 409 | Se intenta anular dos veces el mismo movimiento |
| `VERSION_CONFLICTO` | 412 | Otro usuario modificó el producto primero |
| `DATOS_INVALIDOS` | 422 | Campo obligatorio faltante, cantidad <= 0 o decimales no admitidos por la unidad (RN-02, RN-04) |
| `METODO_NO_PERMITIDO` | 405 | Método inexistente, por ejemplo `DELETE /productos/{id}` (RN-12) |
| `NO_IMPLEMENTADO` | 501 | Endpoint definido en el contrato pero aún sin lógica. Desaparece cuando se implementa (criterio de B12) |

## 5. Catálogo de endpoints por servicio

Son **37 endpoints** de negocio, más los de plataforma. La tabla coincide con los contratos `docs/openapi/*-v1.yaml` (verificado el 01-nov-2026). Cada fila es una funcionalidad del inventario con su verbo CRUD, el permiso que exige y la historia que cumple. Todas las rutas cuelgan de `/api/v1`.

### S1 · Identidad

| Método | Ruta | Funcionalidad | Permiso | Traza |
| --- | --- | --- | --- | --- |
| `POST` | `/auth/login` | Iniciar sesión con correo y contraseña. Devuelve `access_token`, `expires_in` y el usuario con su rol y permisos | Público | HU-01 |
| `POST` | `/auth/recuperar` | Solicitar recuperación de contraseña. Supabase Auth envía el correo. Responde `202` aunque el correo no exista (no revela cuentas) | Público | — |
| `GET` | `/auth/me` | Perfil, rol y permisos de la sesión actual. El frontend lo usa para mostrar solo las opciones autorizadas | Autenticado | HU-01, RNF-07 |
| `GET` | `/usuarios` | Listar usuarios (filtros `activo`, `rol`) | `usuario.gestionar` | HU-09 |
| `POST` | `/usuarios` | Crear usuario con correo, nombre, rol y contraseña temporal | `usuario.gestionar` | HU-09, RF-13 |
| `GET` | `/usuarios/{id}` | Consultar un usuario | `usuario.gestionar` | HU-09 |
| `PATCH` | `/usuarios/{id}` | Cambiar nombre o rol | `usuario.gestionar` | HU-09 |
| `PATCH` | `/usuarios/{id}/estado` | Activar o desactivar (`{"activo": false}`). No existe `DELETE` | `usuario.gestionar` | HU-09 |
| `GET` | `/roles` | Roles con sus permisos, para el formulario de usuarios | `usuario.gestionar` | RF-01 |

### S2 · Catálogo

| Método | Ruta | Funcionalidad | Permiso | Traza |
| --- | --- | --- | --- | --- |
| `GET` | `/categorias` | Listar categorías activas para el filtro | `producto.consultar` | HU-04, RF-12 |
| `POST` | `/categorias` | Crear categoría | `catalogo.administrar` | RNF-09 |
| `PATCH` | `/categorias/{id}` | Renombrar o desactivar categoría | `catalogo.administrar` | RNF-09 |
| `GET` | `/unidades-medida` | Listar las 12 unidades e indicar si admiten decimales | `producto.consultar` | HU-02 |
| `GET` | `/productos` | Consultar inventario vigente con filtros `categoria_id`, `estado`, `q` e `incluir_descontinuados`. Muestra código, nombre, stock, unidad y estado | `producto.consultar` | HU-03, HU-04, RF-03, RF-11, RF-19 |
| `GET` | `/productos/{id}` | Detalle del producto con su `version` | `producto.consultar` | RF-03 |
| `POST` | `/productos` | Crear producto. Si trae `cantidad_inicial`, la registra como movimiento `ENTRADA` con motivo `SALDO_INICIAL` en la misma transacción | `producto.crear` | HU-02, RF-02, RN-01, RN-02 |
| `PATCH` | `/productos/{id}` | Actualizar nombre, descripción, categoría o unidad con `If-Match`. No permite cambiar `codigo` ni `stock_actual` | `producto.editar` | HU-05, RF-04 |
| `PUT` | `/productos/{id}/stock-minimo` | Configurar el stock mínimo; `null` lo quita | `producto.configurar_minimo` | RF-16, RN-14 |
| `POST` | `/productos/{id}/descontinuar` | Marcar como descontinuado, conservando su historia | `producto.descontinuar` | HU-12, RF-05, RN-11, RN-12 |
| `POST` | `/productos/{id}/reactivar` | Revertir la descontinuación | `producto.descontinuar` | HU-12 |

| `GET` | `/proveedores` | Listar proveedores (`?incluir_inactivos`) | `producto.consultar` | RF-17 |
| `POST` | `/proveedores` | Registrar proveedor con su ubicación. NIT único | `catalogo.administrar` | — |
| `PATCH` | `/proveedores/{id}` | Actualizar o desactivar proveedor. No existe `DELETE` | `catalogo.administrar` | — |

No existe `DELETE /productos`: responde `405` (RN-12).

### S3 · Movimientos

| Método | Ruta | Funcionalidad | Permiso | Traza |
| --- | --- | --- | --- | --- |
| `GET` | `/motivos-movimiento` | Motivos válidos por tipo (`?tipo=SALIDA`) | `movimiento.consultar` | RF-10 |
| `POST` | `/movimientos/entradas` | Registrar entrada con `producto_id`, `cantidad`, `motivo_codigo` y, opcionalmente, `costo_unitario` y `observacion`. Devuelve el movimiento, el saldo y el estado nuevo | `movimiento.registrar_entrada` | HU-06, RF-06, RF-09, RF-10 |
| `POST` | `/movimientos/salidas` | Registrar salida o consumo. Responde `409 STOCK_INSUFICIENTE` si supera el disponible | `movimiento.registrar_salida` | HU-07, HU-11, RF-07, RF-08, RF-09 |
| `GET` | `/movimientos` | Historial con filtros `producto_id`, `tipo`, `usuario_id`, `desde` y `hasta`. Muestra fecha, producto, cantidad, tipo y responsable | `movimiento.consultar` | HU-08, RF-14, RF-15 |
| `GET` | `/movimientos/{id}` | Detalle de un movimiento y de su anulación, si existe | `movimiento.consultar` | HU-08 |
| `POST` | `/movimientos/{id}/anulaciones` | Anular con `motivo` y `confirmacion: true`. Crea un movimiento `ANULACION` que revierte el saldo | `movimiento.anular` | HU-13, RF-20, RN-13 |

No existe `DELETE /movimientos`: responde `405`. La anulación reemplaza a la eliminación y conserva la trazabilidad (DEC-06, ADR-0003).

### S4 · Alertas y analítica

| Método | Ruta | Funcionalidad | Permiso | Traza |
| --- | --- | --- | --- | --- |
| `GET` | `/alertas` | Productos que requieren reposición (`?estado=BAJO_STOCK` o `AGOTADO`) con su faltante | `alerta.consultar` | HU-10, HU-11, RF-17, RF-18 |
| `GET` | `/alertas/resumen` | Conteo por estado, para el indicador del menú | `alerta.consultar` | RF-18 |
| `GET` | `/analitica/tablero` | Indicadores de decisión: productos por estado, valor del inventario, consumo de 30 días y los 10 productos de mayor rotación | `analitica.consultar` | RF-19 |
| `GET` | `/analitica/consumo` | Consumo por `categoria`, `producto` o `dia` en un rango de fechas | `analitica.consultar` | RF-14 |
| `GET` | `/analitica/cobertura` | Días de cobertura estimados: stock actual sobre consumo diario promedio de 30 días | `analitica.consultar` | RF-17 |
| `GET` | `/analitica/kardex` | Kardex valorizado de un producto con costo promedio ponderado (`?producto_id`, `desde`, `hasta`) | `analitica.consultar` | RF-14, DEC-11 |
| `GET` | `/analitica/proveedores-cercanos` | Proveedores dentro de un radio, en GeoJSON (`?latitud`, `longitud`, `radio_km`) | `alerta.consultar` | RF-17 |
| `GET` | `/analitica/exportar` | Descargar inventario o movimientos en `csv` o `parquet`, generado con DuckDB | `analitica.consultar` | RNF-10 |

El valor del inventario depende de DEC-11 (costo promedio ponderado), que se ratifica en A1, y de la política contable de A3. Ambas tareas están abiertas.

### Plataforma (en cada servicio)

| Método | Ruta | Funcionalidad |
| --- | --- | --- |
| `GET` | `/health` | Vida del proceso, sin tocar la base. Para Vercel e Istio |
| `GET` | `/health/ready` | Listo para atender: hace `SELECT 1` en la base. Responde `503` con `sin_base` si no la alcanza |
| `GET` | `/docs`, `/openapi.json` | Contrato vivo generado por FastAPI (bajo `/api/v1`) |

## 6. Matriz de permisos por rol

Los 13 permisos se siembran en `seguridad.permiso` y se asignan por rol en `seguridad.rol_permiso`. Las filas marcadas con DEC dependen de una decisión pendiente del grupo; mientras no se decida, rige el valor de la tabla.

| Permiso | Administrador | Manicurista | Gerente (DEC-01) |
| --- | --- | --- | --- |
| `usuario.gestionar` | Sí (DEC-02) | No | Sí |
| `producto.consultar` | Sí | Sí | Sí |
| `producto.crear` | Sí | No | No |
| `producto.editar` | Sí | No | No |
| `producto.configurar_minimo` | Sí | No | No |
| `producto.descontinuar` | Sí | No | No |
| `catalogo.administrar` | Sí | No | No |
| `movimiento.registrar_entrada` | Sí | No (DEC-03) | No |
| `movimiento.registrar_salida` | Sí | Sí | No |
| `movimiento.consultar` | Sí | Solo los propios | Sí |
| `movimiento.anular` | Sí | No | No |
| `alerta.consultar` | Sí | Sí (DEC-04) | Sí |
| `analitica.consultar` | Sí | No | Sí |

El backend valida el permiso en cada endpoint y la base lo vuelve a validar con RLS (`seguridad.tiene_permiso`). El frontend solo **oculta** opciones; nunca es la barrera de seguridad.

## 7. Estructura de los repositorios

Cada servicio vive en su propia carpeta con las 4 capas hexagonales. Lo común (configuración, sesión de base, seguridad, errores) está en `compartido/`, y ningún módulo importa otro módulo.

**`spa-backend-api`:**

```
app/
├── main.py                     # Crea FastAPI e incluye los 4 routers bajo /api/v1
├── compartido/
│   ├── config.py               # Pydantic Settings: DB_URL, JWT_*, CORS_ORIGINS, SUPABASE_*
│   ├── db.py                   # Engine SQLAlchemy (pooler, prepare_threshold=None) y sesión por request
│   ├── seguridad.py            # Emitir y decodificar JWT (PyJWT) y requiere(Permiso.X)
│   ├── permisos.py             # Los 13 permisos
│   ├── errores.py              # Excepciones de dominio → problem+json con code
│   ├── contrato.py             # Respuestas de error comunes y operationId
│   ├── paginacion.py           # ?page&size → {items,total,page,size}
│   ├── tipos.py                # Cantidad y Costo como texto decimal
│   └── salud.py                # /health y /health/ready
├── modulos/
│   ├── identidad/              # S1
│   │   ├── domain/             # Usuario, Rol, Permiso (sin FastAPI ni SQLAlchemy)
│   │   ├── application/        # Casos de uso + puertos (UsuarioRepository, ProveedorIdentidad)
│   │   ├── infrastructure/     # Adaptadores: SQLAlchemy, Supabase Auth
│   │   └── api/                # Router, esquemas Pydantic, dependencias
│   ├── catalogo/               # S2 (misma estructura)
│   ├── movimientos/            # S3 (misma estructura)
│   └── analitica/              # S4 (misma estructura; pandas + DuckDB en infrastructure/)
└── tests/
    ├── contrato/               # Los 37 endpoints: existen, exigen su permiso y responden problem+json
    ├── unit/<modulo>/          # Dominio y casos de uso, sin base
    └── integration/<modulo>/   # Contra PostGIS local (docker compose); ver nota de la sección 8
docs/openapi/
├── identidad-v1.yaml
├── catalogo-v1.yaml
├── movimientos-v1.yaml
└── analitica-v1.yaml
.importlinter                   # Contrato: modulos.* independientes entre sí
```

**`spa-frontend-web`:** una carpeta de funcionalidad por servicio y un cliente HTTP por contrato.

```
src/
├── api/
│   ├── cliente.js              # axios con VITE_API_BASE_URL, Bearer y manejo de problem+json
│   ├── identidad.js            # login(), me(), usuarios...
│   ├── catalogo.js
│   ├── movimientos.js
│   └── analitica.js
├── features/
│   ├── auth/                   # Login, sesión, rutas protegidas por permiso
│   ├── usuarios/
│   ├── inventario/             # Tabla, filtros, detalle y formulario de producto
│   ├── movimientos/            # Entrada, salida, historial, anulación
│   ├── proveedores/            # Proveedores georreferenciados y mapa
│   └── tablero/                # Alertas e indicadores
├── mocks/                      # Modo simulado (VITE_API_MOCK=true): adaptador de Axios con la forma del contrato
└── design-system/              # Tokens de marca, componentes base (Tailwind)
```

## 8. Flujo de trabajo backend → frontend → QA

Primero se aprueba el contrato; después backend y frontend trabajan en paralelo; QA certifica en staging, y solo entonces el cambio llega a producción.

1. **Contrato OpenAPI.** `docs/openapi/<servicio>-v1.yaml`, en un PR hacia `develop`.
2. **Revisión del contrato.** Se aprueba o se piden cambios en el PR (los 4 contratos v1 se cerraron en la tarea B2).
3. **Implementación en paralelo.** Backend: rama `feature/<tarea>-descripcion` con pruebas unitarias y de integración. Frontend: pantallas con el modo simulado (`VITE_API_MOCK=true`, adaptador de axios en `src/mocks`) usando la forma del contrato.
4. **PR a `develop`.** Checks `politica / pr`, `calidad / pipeline` (ruff, mypy, pytest con cobertura ≥ 80 %, pip-audit) y `seguridad / scan`, más la URL Preview de Vercel.
5. **`develop` = staging.** API y web en Vercel; base de staging en Supabase, que actualiza el pipeline `cd`.
6. **Certificación de QA** con `spa-qa` (pytest + httpx para la API, Playwright para E2E) contra staging y con los criterios de cada HU (`spa-qa/docs/PLAN_PRUEBAS.md`).
7. **Release `develop` → `main`.** Aprobación de equipo-qa y merge commit (ADR-0006); despliegue de producción en Vercel y `cd` de base aprobado.

Si QA encuentra un fallo en el paso 6, abre un issue al autor y se vuelve al paso 3.

Un cambio de contrato después del paso 2 exige un PR nuevo de contrato aprobado por frontend.

**Definición de terminado de un PR de backend:**

- [ ] El endpoint coincide con su contrato: `tests/test_contratos.py` compara el código con `docs/openapi/<servicio>-v1.yaml` (se regeneran con `python scripts/exportar_contratos.py`)
- [ ] Pruebas unitarias del dominio y de integración contra PostGIS; cobertura de al menos 80 %. **Nota verificada el 01-nov:** en CI las pruebas de integración se omiten (`skip`), porque `tests/integration/conftest.py` se conecta a `localhost:54322` y el servicio de base del CI no tiene el esquema de `spa-database`. Hasta corregirlo, el PR adjunta la salida de esas pruebas ejecutadas en local
- [ ] Casos de error cubiertos con su `code`: sin token, sin permiso, datos inválidos y reglas de negocio
- [ ] Sin DDL en la API y sin importar otro módulo (`import-linter` en verde)
- [ ] La URL Preview de Vercel responde y el PR la enlaza para que frontend y QA la prueben
- [ ] El PR enlaza su issue: en PR hacia `develop` se escribe `Refs #n` y QA cierra el issue al fusionar; en repositorios sin `develop` (`spa-docs`, `spa-org-bootstrap`) se escribe `Closes #n`

## 9. Paquete por desarrollador

Cada persona recibe tareas concretas (issues con la etiqueta `plan-entregas`), y los endpoints se reparten por tarea para que los PR de dos personas no toquen los mismos archivos al mismo tiempo. Las cifras de esta tabla suman los 37 endpoints.

| Persona | Endpoints por tarea | Otras tareas |
| --- | --- | --- |
| **Damx0611** (backend) · 16 endpoints | B4: `GET /categorias`, `GET /unidades-medida` · B8: `GET /productos`, `GET /productos/{id}` · B9: `POST /productos`, `PATCH /productos/{id}`, `stock-minimo`, `descontinuar`, `reactivar`, `POST` y `PATCH /categorias` · B10: entradas y salidas · B11: historial, detalle y anulación | B3 (dominio del catálogo, cerrada), B12 (cierre del backend), Q5 (prueba de carga) |
| **ivonnelizethmoreno-cloud** (backend y analítica) · 12 endpoints | B5: `POST /auth/login`, `GET /auth/me` · B6: 5 de `/usuarios` y `GET /roles` · B14: tablero, consumo y cobertura · X7: kardex | B1 (entorno local con Docker, reasignada el 31-oct), B2 (contratos, cerrada), A1, A2, A3 |
| **Gleidy725** (frontend) · 4 endpoints | X1: `GET /motivos-movimiento` · X2: `GET`, `POST` y `PATCH /proveedores` | F2 (manual de marca), F3, F5, F10, F11, F12, F13 (pantallas), D11 (datos de demostración) |
| **Estefa44445** (frontend) · 5 endpoints | X3: `POST /auth/recuperar` · X4: `GET /alertas` y `/alertas/resumen` · X5: `GET /analitica/exportar` · X6: `GET /analitica/proveedores-cercanos` | F1 (cerrada), F4, F6, F8, F15, F16, F17 |
| **mikeforeros** (datos) | — | D0, D2 y D3 (cerradas; D2 y D3 las adelantó QA), D1, D5 a D10, D12, D13 |
| **jhonjairosolerumbarila-ops** (QA y DevOps) | — | J0, J1, J2, D4, Q1 a Q4, DV3 a DV7, R1 a R3 |

**Fechas del plan:** Entrega 1 el 31-oct (identidad y catálogo). La release 0.1.0 (R1) se movió al 01-nov; depende de Q2 y DV3, y sus condiciones están escritas en el issue spa-backend-api#28. Entrega 2 el 06-nov (movimientos, alertas, analítica, geografía, Kubernetes con Istio y demostración en EKS). Release 1.0.0 el 07-nov y entrega al cliente el 08-nov.

## 10. Configuración por entorno y ruta a AWS

Las variables se cargan en Vercel. Ninguna va en el repositorio; `.env.example` lista solo los nombres. **La columna de staging refleja la configuración hecha entre el 26 y el 29 de octubre y no se reverificó en esta revisión. La de producción está pendiente de DV3.**

| Variable | Tipo en Vercel | Servicios | Valor en staging (Preview) | Producción |
| --- | --- | --- | --- | --- |
| `APP_ENV` | Config | Todos | `staging` | `production` |
| `DB_URL` | Secret | Todos | Pooler de transacciones del proyecto de staging, puerto 6543, `postgresql+psycopg://...?sslmode=require` | Pendiente: DV3 |
| `JWT_SECRET` | Secret | S1 la firma; todos la verifican | Cargada | Pendiente: DV3 (distinta a la de staging) |
| `JWT_ALGORITHM`, `JWT_EXPIRES_MINUTES` | Config | Todos | `HS256`, `30` | Igual |
| `CORS_ORIGINS` | Config | Todos | Cargada | Pendiente: DV3 (solo el dominio web de producción) |
| `SUPABASE_URL` | Config | S1 | `https://fphsadutyrynmbqewxnd.supabase.co` | Pendiente: DV3 |
| `SUPABASE_SECRET_KEY` | Secret | Solo S1 | Clave secreta del proyecto, para crear usuarios en Supabase Auth | Pendiente: DV3 |
| `VITE_API_BASE_URL` | Config (frontend) | Web | API de staging `/api/v1` | API de producción `/api/v1` |

`SUPABASE_SECRET_KEY` salta RLS: solo el adaptador de Supabase Auth de S1 puede leerla, nunca otro módulo ni el frontend.

**Kubernetes y AWS (Entrega 2).** Estado verificado en `spa-infra-aws` el 01-nov:

- Existen manifiestos para **una** API (`k8s/base/backend`: `deployment-v1`, `deployment-v2`, `hpa`, `service`) y para Istio (`k8s/istio`: `PeerAuthentication`, `AuthorizationPolicy`, `DestinationRule` y `VirtualService` de canario).
- Las imágenes de esos manifiestos apuntan hoy a ECR con un marcador (`ACCOUNT.dkr.ecr...`). DV5 exige imágenes publicadas en GHCR (DV4) y ejecutar también la web; ese cambio está pendiente.
- DV5 (clúster local con kind, Istio con mTLS STRICT y canario 90/10) y DV7 (EKS con Terraform, con evidencia y `terraform destroy` el mismo día) están abiertas.

**Evolución posterior, fuera del plan de entregas.** Si cada carpeta `modulos/<servicio>` se separara en un repositorio `spa-svc-<servicio>` creado desde `spa-template-servicio`, el Ingress Gateway de Istio enrutaría por prefijo sin que el frontend cambie su `VITE_API_BASE_URL`:

| Prefijo bajo `/api/v1` | Servicio |
| --- | --- |
| `/auth`, `/usuarios`, `/roles` | `spa-svc-identidad` |
| `/productos`, `/categorias`, `/unidades-medida`, `/proveedores` | `spa-svc-catalogo` |
| `/movimientos`, `/motivos-movimiento` | `spa-svc-movimientos` |
| `/alertas`, `/analitica` | `spa-svc-analitica` |

## 11. Ampliación del alcance y reparto full-stack (plan de 2 entregas)

Para cubrir todo el alcance antes de la entrega al cliente (08-nov), el contrato pasa de 31 a **37 endpoints**. Los dos integrantes de frontend construyen 9 endpoints de punta a punta (endpoint y pantalla), lo que equilibra la carga del equipo.

| Endpoint nuevo | Servicio | Permiso | Responsable · tarea | Base en S0 |
| --- | --- | --- | --- | --- |
| `POST /auth/recuperar` (202) | S1 | Público | Estefa44445 · X3 | Supabase Auth |
| `GET`, `POST`, `PATCH /proveedores` | S2 | `producto.consultar` (GET); `catalogo.administrar` (POST, PATCH) | Gleidy725 · X2 | `inventario.proveedor` con `geography(Point, 4326)` e índice GIST (D12) |
| `GET /analitica/kardex` | S4 | `analitica.consultar` | ivonnelizethmoreno-cloud · X7 | Costo promedio ponderado (NIIF para Pymes, Sección 13 · DEC-11) |
| `GET /analitica/proveedores-cercanos` (GeoJSON) | S4 | `alerta.consultar` | Estefa44445 · X6 | `inventario.proveedores_cercanos()` con `ST_DWithin` (D12) |

Endpoints existentes que pasan al frontend: `GET /motivos-movimiento` (Gleidy725 · X1), `GET /alertas` y `/alertas/resumen` (Estefa44445 · X4), `GET /analitica/exportar` (Estefa44445 · X5).

Las entradas por `COMPRA` llevan `costo_unitario` y `proveedor_id`; el trigger de S0 (D7) recalcula el costo promedio, y con eso el tablero puede calcular `valor_inventario`.

Las fechas y el seguimiento están en la sección 9. El seguimiento lo hace QA con comentarios en los issues: no hay un workflow automático de avisos (verificado el 31-oct: `spa-org-bootstrap` solo tiene los workflows `ci`, `Dependabot Updates` y `Dependency Graph`).
