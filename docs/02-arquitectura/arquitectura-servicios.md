# Arquitectura de servicios y contratos de API · Spa de Uñas

Oct 25, 2026 · @Jhon Jairo Soler

> **Copia en el repositorio** de la revisión 15 del documento de arquitectura (redactado el 25-oct-2026), exportada el 01-nov-2026.
>
> Diferencias conocidas con el estado actual del proyecto, verificadas el 01-nov-2026:
>
> 1. Sección 3: `respaldo.yml` y `keep-alive.yml` ya existen en `spa-database/.github/workflows/` (ya no están «por crear»).
> 2. Sección 3: las categorías y unidades llegan por la migración `20261031130000_catalogo_datos.sql`, no por el seed. `seed.sql` se ejecuta una sola vez en Supabase y queda solo para datos de demostración.
> 3. Secciones 1 y 2: el ADR-0009 citado no está en `docs/02-arquitectura/adr/` (allí hay de 0000 a 0006).
> 4. Sección 8: el diagrama del flujo de trabajo no se pudo exportar a Markdown.
> 5. Sección 11: el «workflow diario» de avisos no está instalado; los recordatorios los hace QA.
>
> Si hay contradicción, mandan el código, los contratos (`spa-backend-api/docs/openapi/`) y las migraciones (`spa-database/supabase/migrations/`).


## 1. Decisión de arquitectura

El backend se divide en **4 servicios de dominio** (Identidad, Catálogo, Movimientos y Alertas y analítica) más **S0 · Datos**, el servicio de plataforma que recibe la base de datos. En los Sprints 1 a 3 los 4 servicios de dominio se despliegan juntos como **monolito modular** en un solo proyecto de Vercel (`spa-unas-api`), mientras S0 se despliega aparte con su propio pipeline. En el Sprint 4 cada servicio de dominio se extrae a su propio microservicio (`spa-svc-<dominio>`) en EKS con Istio, sin cambiar las rutas que consume el frontend.

Por qué así y no con microservicios desde el primer día:

- **Equipo pequeño.** Hay 2 desarrolladores de backend. Cuatro repositorios, cuatro pipelines y cuatro proyectos de Vercel multiplicarían el trabajo de operación sin agregar funcionalidad.
- **Consistencia.** Una salida de inventario debe bloquear el producto y descontar el stock en la **misma transacción** (RN-05, RN-06). Ese invariante lo garantiza la base con un trigger, no la red entre servicios.
- **Extracción sin reescritura.** Cada módulo ya tiene sus propias capas hexagonales, su contrato OpenAPI, su prefijo de rutas y sus tablas. Extraerlo es copiar la carpeta a un repositorio creado desde `spa-template-servicio` y agregar una ruta en Istio.
- **Reglas verificables.** Un módulo **nunca** importa el código interno de otro. Lo comprueba `import-linter` en el check `calidad / pipeline`, así que la frontera se puede auditar desde el primer commit.

Esta decisión se registra como **ADR-0009 · Monolito modular con 4 servicios de dominio**.

## 2. Mapa de servicios

Cada servicio es dueño de un conjunto de funcionalidades, de sus rutas y de sus tablas. Las 13 historias de usuario y los 20 requerimientos funcionales quedan repartidos sin solaparse.

| Servicio | Responsabilidad | Historias | Requerimientos | Prefijos de ruta | Hito |
| --- | --- | --- | --- | --- | --- |
| **S0 · Datos** | Recibe la base de datos: esquema, migraciones, seeds, roles y permisos de base, vistas de consulta, respaldos y despliegue | Todas (soporte) | Reglas de negocio que se garantizan en la base, RNF-06, RNF-10 | Sin rutas HTTP: su contrato es el esquema versionado | Sprint 1 (D1 a D4) |
| **S1 · Identidad** | Autenticación, emisión de JWT, usuarios, roles y permisos | HU-01, HU-09 | RF-01, RF-13 | `/auth`, `/usuarios`, `/roles` | M1 |
| **S2 · Catálogo** | Productos, categorías, unidades, stock mínimo, descontinuación y estado del inventario | HU-02 a HU-05, HU-12 | RF-02 a RF-05, RF-11, RF-12, RF-16, RF-19 | `/productos`, `/categorias`, `/unidades-medida` | M2 |
| **S3 · Movimientos** | Entradas, salidas, validación de stock, kardex, historial y anulación controlada | HU-06 a HU-08, HU-11, HU-13 | RF-06 a RF-10, RF-14, RF-15, RF-20 | `/movimientos`, `/motivos-movimiento` | M3 |
| **S4 · Alertas y analítica** | Alertas de bajo stock y agotados, tablero de decisiones, consumo, cobertura y exportación | HU-10, HU-11 (consulta) | RF-17, RF-18 | `/alertas`, `/analitica` | M4 |

Todas las rutas HTTP cuelgan de `/api/v1`. La base de datos la recibe **S0 · Datos** (`spa-database`, a cargo de mikeforeros): crea y versiona el esquema y lo despliega con su propio pipeline. S1 a S4 se conectan solo a sus propias tablas (sección 3).

La **extensión geográfica** (varias sedes o bodegas con ubicación en PostGIS) queda en el backlog. La especificación describe un solo establecimiento, así que requiere una decisión nueva (DEC-12) antes de construirse. PostGIS ya está instalado en la migración base.

## 3. Propiedad de los datos y acceso a la base

La base de datos la recibe y administra **S0 · Datos**, el repositorio `spa-database`. Mike escribe las migraciones y el pipeline `cd` las aplica en Supabase. Los servicios S1 a S4 nunca ejecutan DDL: solo leen y escriben las tablas que les pertenecen.

### S0 · Servicio de Datos

S0 es un **servicio de plataforma**. Tiene repositorio, responsable, pruebas y pipeline de despliegue propios, y se publica en su propio ciclo. Su contrato con los demás servicios es el **esquema versionado** (tablas, vistas y roles), no una API HTTP.

| Componente | Qué entrega | Dónde vive | Responsable |
| --- | --- | --- | --- |
| Migraciones y seeds | Tablas, restricciones, triggers y catálogos base | `supabase/migrations/`, `supabase/seed.sql` | mikeforeros |
| Pruebas de base | Pruebas SQL en CI sobre un PostGIS limpio, antes de cada fusión | `tests/sql/`, `ci.yml` | mikeforeros |
| Despliegue | `develop` → staging automático; `main` → producción con aprobación de QA | `.github/workflows/cd.yml` | Jhon (pipeline), mikeforeros (contenido) |
| Roles y permisos | Un rol por servicio con `GRANT` mínimo, y RLS | Migraciones | mikeforeros; Jhon custodia las contraseñas |
| Vistas de consulta | `v_inventario` y `v_alerta_stock`, el contrato de lectura para S2 y S4 | Migraciones | mikeforeros |
| Respaldos | `pg_dump` semanal guardado como artefacto | `respaldo.yml` (por crear) | Jhon |
| Disponibilidad | Keep-alive para que el plan Free de Supabase no pause el proyecto | `keep-alive.yml` (por crear) | Jhon |
| Detección de deriva | Comparar el esquema de staging con las migraciones (`supabase db diff`) | Workflow por crear | mikeforeros |

Por qué S0 no expone un CRUD por HTTP para que los otros servicios lean y escriban a través de él:

- **Atomicidad.** Registrar una salida y descontar el stock debe ocurrir en una sola transacción. Con un servicio de datos intermedio serían dos llamadas de red, y una podría fallar después de la otra.
- **Latencia.** En Vercel cada llamada entre funciones suma un arranque en frío. Cada operación del inventario pagaría el viaje dos veces.
- **Acoplamiento.** Si ese servicio cae, caen los cuatro dominios. Sería un punto único de falla, justo lo que los microservicios buscan evitar.
- **Duplicación.** Las reglas de negocio quedarían repartidas entre el dominio y el servicio de datos.

Si en el futuro alguien necesita leer datos desde afuera (por ejemplo QGIS o ArcGIS para la extensión geográfica, o un tablero externo), S0 lo publica como **vistas de solo lectura**, o mediante la Data API de Supabase con RLS. Nunca como un CRUD de escritura.

| Tabla o vista (esquema) | Dueño que escribe | Puede leer | Regla |
| --- | --- | --- | --- |
| `auth.users` (Supabase Auth) | S1, por la API de Supabase Auth | S1 | Nadie la toca por SQL |
| `seguridad.rol`, `permiso`, `rol_permiso` | Seeds de `spa-database` | S1 | Los permisos cambian por migración, no por la API |
| `seguridad.perfil` | S1 | S1; los demás, solo el nombre del responsable | Desactivar en lugar de borrar |
| `inventario.categoria`, `unidad_medida` | S2 | Todos | Catálogo base en el seed D3 |
| `inventario.producto` | S2, excepto `stock_actual` y `estado` | Todos | `stock_actual` solo cambia por el trigger de movimientos; `estado` es columna generada |
| `inventario.motivo_movimiento` | Seeds de `spa-database` | S3 |  |
| `inventario.movimiento` | S3, solo `INSERT` | S3, S4 | Libro mayor inmutable: se corrige con `ANULACION`, nunca con `UPDATE` ni `DELETE` |
| `inventario.v_inventario`, `v_alerta_stock` | Nadie (vistas) | S2, S4 | `security_invoker = true` |
| `auditoria.registro_cambio` | Solo triggers | S4, con permiso de auditoría | Solo inserción |

**Conexión por fase:**

- **Sprints 1 a 3 (Vercel).** Una sola variable `DB_URL` hacia el pooler de transacciones de Supabase (puerto 6543), con `prepare_threshold=None` en psycopg. La propiedad de las tablas se garantiza con revisión de código y con `import-linter`: cada módulo tiene su propio repositorio SQLAlchemy y no comparte modelos con otro módulo.
- **Sprint 4 (AWS).** Un rol de base por servicio (`svc_identidad`, `svc_catalogo`, `svc_movimientos`, `svc_analitica`) con `GRANT` solo sobre sus tablas, y `svc_analitica` en solo lectura. Mike crea los roles por migración y Jhon custodia sus contraseñas como secretos.
- **Consistencia entre servicios.** Cuando S3 registra una salida, el trigger `trg_movimiento_aplicar` bloquea el producto (`FOR UPDATE`), valida el stock y lo actualiza en la misma transacción. S2 nunca recalcula el stock: lo lee.

## 4. Convenciones del contrato API

El contrato se escribe primero (contract-first) y es la única fuente de verdad entre backend y frontend. Estas convenciones son obligatorias en los 4 servicios.

| Tema | Regla |
| --- | --- |
| Base y versión | `/api/v1/...`. Un cambio incompatible crea `/api/v2`; la versión anterior se mantiene al menos un sprint |
| Recursos | Sustantivos en plural y kebab-case (`/unidades-medida`). Las acciones de negocio son subrecursos con `POST` (`/productos/{id}/descontinuar`) |
| JSON | Campos en `snake_case`, igual que la base. Las cantidades y los costos viajan como **texto decimal** (`"12.500"`) para no perder precisión. Las fechas en ISO 8601 con zona (`2026-10-25T18:30:00-05:00`) |
| Paginación | `?page=1&size=20` (máximo 100). Respuesta `{ "items": [], "total": 0, "page": 1, "size": 20 }` |
| Filtros y orden | Por query string (`?categoria_id=3&estado=BAJO_STOCK&orden=-ocurrido_en`) |
| Errores | `application/problem+json`: `{ type, title, status, detail, code, errors[] }`. El campo `code` es estable y es el que usa el frontend |
| Autenticación | `Authorization: Bearer <jwt>` en todo excepto `/auth/login` y `/health`. JWT HS256 de 30 minutos con `sub`, `rol` y `permisos` |
| Autorización | Cada endpoint declara su permiso con `Depends(requiere("producto.crear"))`. Sin permiso responde `403` aunque se llame directamente (RNF-07) |
| Idempotencia | Los `POST` de movimientos exigen la cabecera `Idempotency-Key` (UUID). Un reintento con la misma clave devuelve el mismo movimiento |
| Concurrencia | `PATCH /productos/{id}` exige `If-Match: <version>`. Si la versión no coincide responde `412` |
| Documentación viva | `/docs` (Swagger) y `/openapi.json`, generados por FastAPI |

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

## 5. Catálogo de endpoints por servicio

Son 31 endpoints de negocio, más 3 de plataforma. Cada fila es una funcionalidad del inventario con su verbo CRUD, el permiso que exige y la historia que cumple. Todas las rutas cuelgan de `/api/v1`.

### S1 · Identidad

| Método | Ruta | Funcionalidad | Permiso | Traza |
| --- | --- | --- | --- | --- |
| `POST` | `/auth/login` | Iniciar sesión con correo y contraseña. Devuelve `access_token`, `expires_in` y el usuario con su rol y permisos | Público | HU-01 |
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
| `GET` | `/analitica/exportar` | Descargar inventario o movimientos en `csv` o `parquet`, generado con DuckDB | `analitica.consultar` | RNF-10 |

El valor del inventario depende de DEC-11 (costo promedio ponderado). Mientras no se decida, el tablero lo muestra como “no disponible”.

### Plataforma (en cada servicio)

| Método | Ruta | Funcionalidad |
| --- | --- | --- |
| `GET` | `/health` | Vida del proceso, sin tocar la base. Para Vercel e Istio |
| `GET` | `/health/ready` | Listo para atender: hace `SELECT 1` en la base |
| `GET` | `/docs`, `/openapi.json` | Contrato vivo generado por FastAPI |

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
│   ├── seguridad.py            # Decodificar JWT (PyJWT) y requiere("permiso")
│   └── errores.py              # Excepciones de dominio → problem+json con code
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
    ├── unit/<modulo>/          # Dominio y casos de uso, sin base
    └── integration/<modulo>/   # Contra PostGIS del servicio de CI
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
│   ├── cliente.js              # fetch con VITE_API_BASE_URL, Bearer y manejo de problem+json
│   ├── identidad.js            # login(), me(), usuarios...
│   ├── catalogo.js
│   ├── movimientos.js
│   └── analitica.js
├── features/
│   ├── auth/                   # Login, sesión, rutas protegidas por permiso
│   ├── usuarios/
│   ├── inventario/             # Tabla, filtros, detalle y formulario de producto
│   ├── movimientos/            # Entrada, salida, historial, anulación
│   └── tablero/                # Alertas e indicadores
├── mocks/                      # Modo simulado (VITE_API_MOCK=true): adaptador de Axios con la forma del contrato
└── design-system/              # Tokens de marca, componentes base (Tailwind)
```

## 8. Flujo de trabajo backend → frontend → QA

Primero se aprueba el contrato; después backend y frontend trabajan en paralelo; QA certifica en staging, y solo entonces el cambio llega a producción.

*[Diagrama del flujo de trabajo (7 pasos, 1 retorno): no se pudo exportar; ver la nota al inicio.]*

Si QA encuentra un fallo, abre un issue al autor y el ciclo vuelve a la implementación. Un cambio de contrato después del paso 2 exige un PR nuevo de contrato aprobado por frontend.

**Definición de terminado de un PR de backend:**

- [ ] El endpoint coincide con su contrato. El check compara `/openapi.json` con `docs/openapi/<servicio>-v1.yaml` y falla ante un cambio incompatible
- [ ] Pruebas unitarias del dominio y de integración contra PostGIS; cobertura de al menos 80 %
- [ ] Casos de error cubiertos con su `code`: sin token, sin permiso, datos inválidos y reglas de negocio
- [ ] Sin DDL en la API y sin importar otro módulo (`import-linter` en verde)
- [ ] La URL Preview de Vercel responde y el PR la enlaza para que frontend y QA la prueben
- [ ] El PR referencia su issue (`Closes #n`) y la historia de usuario que cumple

## 9. Paquete por desarrollador

Cada persona recibe un servicio o una capa completa. Así los PR de dos personas no tocan los mismos archivos.

| Persona | Recibe | Configura | Entrega |
| --- | --- | --- | --- |
| **Damx0611** (backend) | S2 Catálogo y S3 Movimientos | `modulos/catalogo` y `modulos/movimientos`; `compartido/db.py` (engine con pooler) | Los 17 endpoints de S2 y S3 con pruebas unitarias y de integración; cobertura de al menos 80 % |
| **ivonnelizethmoreno-cloud** (backend y analítica) | Contratos OpenAPI, S1 Identidad y S4 Alertas y analítica | `docs/openapi/*.yaml`; `modulos/identidad` y `modulos/analitica`; `compartido/seguridad.py` y `errores.py` | Los 4 contratos aprobados por frontend antes de cada sprint; los 14 endpoints de S1 y S4; consultas analíticas con pandas y DuckDB |
| **Gleidy725** (frontend) | Pantallas de inventario y movimientos | `src/api/catalogo.js` y `movimientos.js`; `features/inventario` y `features/movimientos`; casos del modo simulado (src/mocks) de esos contratos | Inventario con filtros, detalle y formulario de producto; entrada, salida, historial y anulación con confirmación |
| **Estefa44445** (frontend) | Identidad visual, acceso y tablero | `design-system/`; `src/api/cliente.js` e `identidad.js`; `features/auth`, `usuarios` y `tablero` | Manual de marca en tokens Tailwind; login y rutas protegidas por permiso; gestión de usuarios; alertas e indicadores |
| **mikeforeros** (datos) | `spa-database` | `supabase/migrations`, `seed.sql`, `tests/sql` | Esquemas `seguridad`, `inventario` y `auditoria`; triggers de stock; RLS; seed del catálogo y de los permisos; vistas para S4; roles por servicio en el Sprint 4 |
| **Jhon** (QA y DevOps) | Plataforma, pipelines y certificación | Entornos de Vercel y GitHub, secretos, `spa-qa` | Suite de API (pytest y httpx) y E2E (Playwright) contra staging; aprobación de cada release a `main` |

Antes de que Damx0611 empiece hay que completar su ingreso a la organización y a `equipo-backend`. Sin ese acceso, GitHub descarta en silencio las asignaciones de issues.

**Orden de entrega por sprint** (alineado con los hitos M1 a M4):

1. **Sprint 1:** contrato `catalogo-v1` y `GET /categorias` de punta a punta (esqueleto andante).
2. **Sprint 2:** S1 Identidad completo y el CRUD de productos de S2. Desde aquí, todos los endpoints exigen JWT.
3. **Sprint 3:** S3 Movimientos y `/alertas` de S4.
4. **Sprint 4:** `/analitica` de S4 y extracción de los 4 servicios a EKS con Istio.

## 10. Configuración por entorno y ruta a AWS

Las variables se cargan en Vercel (Sprints 1 a 3) y en Secrets de Kubernetes (Sprint 4). Ninguna va en el repositorio; `.env.example` lista solo los nombres.

| Variable | Tipo en Vercel | Servicios | Valor en staging (Preview) | Producción |
| --- | --- | --- | --- | --- |
| `APP_ENV` | Config | Todos | `staging` | `production` |
| `DB_URL` | Secret | Todos | Pooler de `spa-unas-staging`, puerto 6543, `postgresql+psycopg://...?sslmode=require` | Pooler de `spa-unas-prod`, creado antes del primer release |
| `JWT_SECRET` | Secret | S1 la firma; todos la verifican | Ya cargada | Una distinta a la de staging |
| `JWT_ALGORITHM`, `JWT_EXPIRES_MINUTES` | Config | Todos | `HS256`, `30` | Igual |
| `CORS_ORIGINS` | Config | Todos | Ya cargada (localhost, web de producción y web de `develop`) | Solo el dominio web de producción |
| `SUPABASE_URL` | Config | S1 | `https://fphsadutyrynmbqewxnd.supabase.co` | URL de `spa-unas-prod` |
| `SUPABASE_SECRET_KEY` | Secret | Solo S1 | Clave secreta del proyecto, para crear usuarios en Supabase Auth | La de producción |
| `VITE_API_BASE_URL` | Config (frontend) | Web | API de staging `/api/v1` | API de producción `/api/v1` |

`SUPABASE_SECRET_KEY` salta RLS: solo el adaptador de Supabase Auth de S1 puede leerla, nunca otro módulo ni el frontend.

**Ruta a AWS (Sprint 4).** Cada carpeta `modulos/<servicio>` pasa a un repositorio `spa-svc-<servicio>` creado desde `spa-template-servicio`. El Ingress Gateway de Istio publica el mismo dominio y enruta por prefijo, así que el frontend no cambia su `VITE_API_BASE_URL`:

| Prefijo bajo `/api/v1` | Servicio en EKS | Réplicas mínimas |
| --- | --- | --- |
| `/auth`, `/usuarios`, `/roles` | `spa-svc-identidad` | 2 |
| `/productos`, `/categorias`, `/unidades-medida` | `spa-svc-catalogo` | 2 |
| `/movimientos`, `/motivos-movimiento` | `spa-svc-movimientos` | 2 |
| `/alertas`, `/analitica` | `spa-svc-analitica` | 1 |

Entre servicios, Istio exige mTLS (`PeerAuthentication` en modo STRICT) y una `AuthorizationPolicy` por servicio. Cada versión nueva se libera como canario (`VirtualService` con 90/10), como ya está preparado en `spa-infra-aws/k8s/istio`.

## 11. Ampliación del alcance y reparto full-stack (plan de 2 entregas)

Para cubrir todo el alcance antes de la entrega al cliente (08-nov), el contrato pasa de 31 a **37 endpoints**. Los dos integrantes de frontend construyen 9 endpoints de punta a punta (endpoint y pantalla), lo que equilibra la carga del equipo.

| Endpoint nuevo | Servicio | Responsable | Base en S0 |
| --- | --- | --- | --- |
| `POST /auth/recuperar` (público, 202) | S1 | Estefa44445 | Supabase Auth |
| `GET`, `POST`, `PATCH /proveedores` | S2 | Gleidy725 | `inventario.proveedor` con `geography(Point, 4326)` e índice GIST |
| `GET /analitica/kardex` | S4 | ivonnelizethmoreno-cloud | Costo promedio ponderado (NIIF para Pymes, Sección 13 · DEC-11) |
| `GET /analitica/proveedores-cercanos` (GeoJSON) | S4 | Estefa44445 | `inventario.proveedores_cercanos()` con `ST_DWithin` |

Endpoints existentes que pasan al frontend: `GET /motivos-movimiento` (Gleidy725), `GET /alertas` y `/alertas/resumen` (Estefa44445), `GET /analitica/exportar` (Estefa44445).

Las entradas por `COMPRA` llevan `costo_unitario` y `proveedor_id`; el trigger de S0 recalcula el costo promedio, y `valor_inventario` del tablero deja de ser nulo.

Fechas límite: Entrega 1 el 31-oct (identidad y catálogo, release 0.1.0) y Entrega 2 el 06-nov (movimientos, alertas, analítica, geografía, Kubernetes con Istio y despliegue demostrativo en EKS). El 07-nov se publica la release 1.0.0 y el 08-nov se hace la entrega al cliente. Cada tarea es un issue con la etiqueta `plan-entregas`. Un workflow diario avisa al responsable, con copia a QA, el día antes de cada fecha límite, el mismo día y si la tarea se vence.
