# Matriz de trazabilidad HU ↔ requisitos ↔ repositorios

| HU | Nombre | RF / RNF / RN | Milestone | DB | API | Web | QA |
|---|---|---|---|:-:|:-:|:-:|:-:|
| HU-01 | Autenticación | RF-01 · RNF-01 · RNF-07 | M1 | ● | ● | ● | ● |
| HU-02 | Registrar producto | RF-02 · RF-16 · RN-01 · RN-02 | M2 | ● | ● | ● | ● |
| HU-03 | Consultar inventario | RF-03 · RF-11 · RF-19 · RNF-08 | M2 | ● | ● | ● | ● |
| HU-04 | Filtrar por categoría | RF-12 | M2 |  | ● | ● | ● |
| HU-05 | Actualizar producto | RF-04 · RN-02 | M2 |  | ● | ● | ● |
| HU-06 | Registrar entrada | RF-06 · RF-09 · RF-10 · RN-03 · RN-04 · RN-07 · RNF-06 | M3 | ● | ● | ● | ● |
| HU-07 | Registrar salida | RF-07 · RF-08 · RF-09 · RF-10 · RN-03..RN-07 · RNF-02 | M3 | ● | ● | ● | ● |
| HU-08 | Consultar historial | RF-14 · RF-15 · RNF-06 | M3 | ● | ● | ● | ● |
| HU-09 | Gestionar usuarios | RF-01 · RF-13 · RNF-07 · RNF-09 | M1 | ● | ● | ● | ● |
| HU-10 | Alertas de bajo stock | RF-16 · RF-17 · RF-18 · RN-09 · RN-14 | M4 | ● | ● | ● | ● |
| HU-11 | Productos agotados | RF-08 · RF-19 · RN-05 · RN-10 | M3 | ● | ● | ● | ● |
| HU-12 | Descontinuar producto | RF-05 · RN-11 · RN-12 | M2 | ● | ● | ● | ● |
| HU-13 | Eliminar movimiento | RF-20 · RN-13 · RNF-06 | M4 | ● | ● | ● | ● |

RNF transversales: RNF-03 (usabilidad, Web), RNF-04/05 (rendimiento/disponibilidad, DEC-07),
RNF-10 (respaldo, DEC-09, `spa-infra-aws` + Supabase PITR).

## Trazabilidad HU ↔ endpoint ↔ pantalla ↔ prueba (A2)

Tarea A2 (spa-docs#32). Fuentes verificadas el 07-nov-2026 (contratos y rutas en `develop`; estado según los PR de cada repositorio):

- **Endpoints:** contratos `spa-backend-api/docs/openapi/*-v1.yaml`. Cada operación declara su HU en la descripción.
- **Pantallas:** rutas de `spa-frontend-web/src/App.jsx` y `src/constants/routes.js`, y la tabla de HU de cada `src/features/*/README.md`.
- **Pruebas:** casos CP del plan de pruebas Q1 (`spa-qa/docs/PLAN_PRUEBAS.md`, sección 3). Los casos 401/403 de cada endpoint están en su sección 4.

**Estado al 07-nov-2026:**

- Backend (`develop`): 2 de los 37 endpoints implementados, `GET /productos` y `GET /productos/{id}` (B8, spa-backend-api#43). Los otros 35 responden `NoImplementado` (501). `POST /auth/login` y `GET /auth/me` están en el PR spa-backend-api#45 (B5), en revisión de QA.
- Frontend (`develop`): pantalla de inicio de sesión implementada (F6, spa-frontend-web#30) con los componentes base de F4 (#23). Las demás páginas son la estructura base del andamiaje. La pantalla de inventario está en el PR spa-frontend-web#25, en revisión de QA.
- La columna «Estado al 07-nov» se actualiza con cada PR fusionado; la trazabilidad queda completa en R2.

| HU | Endpoint (contrato) | Pantalla (ruta · componente) | Pruebas (Q1) | Estado al 07-nov |
|---|---|---|---|---|
| HU-01 Autenticación | `POST /auth/login` · `GET /auth/me` (identidad-v1) | `/login` · `LoginPage`; menú según permisos (`MENU`) | CP-HU01-01 a CP-HU01-08 | Pantalla en develop (F6). API en PR #45 (B5) |
| HU-02 Registrar producto | `POST /productos` · `GET /unidades-medida` (catalogo-v1) | `/inventario` · `InventarioPage` (formulario de producto) | CP-HU02-01 a CP-HU02-06 | API pendiente (B9). Pantalla en PR #25 |
| HU-03 Consultar inventario | `GET /productos` · `GET /productos/{id}` (catalogo-v1) | `/inventario` · `InventarioPage` (tabla con filtros) | CP-HU03-01 a CP-HU03-07 | API en develop (B8). Pantalla en PR #25 |
| HU-04 Filtrar por categoría | `GET /categorias` · `GET /productos?categoria_id=` (catalogo-v1) | `/inventario` · `InventarioPage` (filtro de categoría) | CP-HU04-01, CP-HU04-02 | `GET /productos?categoria_id` en develop (B8); `GET /categorias` pendiente (B4). Pantalla en PR #25 |
| HU-05 Actualizar producto | `PATCH /productos/{id}` con `If-Match` (catalogo-v1) | `/inventario` · `InventarioPage` (edición con `version`) | CP-HU05-01 a CP-HU05-03 | API pendiente (B9). Pantalla en PR #25 |
| HU-06 Registrar entrada | `POST /movimientos/entradas` (movimientos-v1) | `/movimientos` · `MovimientosPage` | CP-HU06-01 a CP-HU06-04 | Pendiente (B10, F12) |
| HU-07 Registrar salida | `POST /movimientos/salidas` (movimientos-v1) | `/movimientos` · `MovimientosPage` | CP-HU07-01 a CP-HU07-03 | Pendiente (B10, F12) |
| HU-08 Consultar historial | `GET /movimientos` · `GET /movimientos/{id}` (movimientos-v1) | `/historial` · `HistorialPage` | CP-HU08-01 a CP-HU08-03 | Pendiente (B11, F13) |
| HU-09 Gestionar usuarios | `GET`/`POST /usuarios` · `GET`/`PATCH /usuarios/{id}` · `PATCH /usuarios/{id}/estado` (identidad-v1) | `/usuarios` · `UsuariosPage` | CP-HU09-01 a CP-HU09-07 | Pendiente (B6, F8) |
| HU-10 Alertas de bajo stock | `GET /alertas` (analitica-v1) | `/tablero` · `TableroPage` (lista de alertas) | CP-HU10-01 a CP-HU10-03 | Pendiente (X4) |
| HU-11 Productos agotados | `POST /movimientos/salidas` · `GET /alertas` | `/movimientos` y `/tablero` | CP-HU11-01 | Pendiente (B10, X4) |
| HU-12 Descontinuar producto | `POST /productos/{id}/descontinuar` · `POST /productos/{id}/reactivar` (catalogo-v1) | `/inventario` · `InventarioPage` | CP-HU12-01 a CP-HU12-03 | API pendiente (B9). Pantalla en PR #25 |
| HU-13 Anular movimiento | `POST /movimientos/{id}/anulaciones` (movimientos-v1). No existe `DELETE` (DEC-06) | `/historial` · `HistorialPage` (anulación con confirmación) | CP-HU13-01 a CP-HU13-03 | Pendiente (B11, F13) |

### Endpoints que no dependen de una sola HU

| Endpoint (contrato) | Requisito | Pantalla | Pruebas (Q1) |
|---|---|---|---|
| `POST /auth/recuperar` (identidad-v1) | HU-01 · DEC-08 | `/recuperar` · `RecuperarPage` | CP-REC-01 |
| `GET /roles` (identidad-v1) | RF-01 | `/usuarios` · `UsuariosPage` | CP-HU09-07 |
| `PUT /productos/{id}/stock-minimo` (catalogo-v1) | RF-16 · RN-14 | `/inventario` · `InventarioPage` | CP-RF16-01 |
| `POST`/`PATCH /categorias` (catalogo-v1) | RNF-09 | `/inventario` | CP-RNF09-01 |
| `GET`/`POST /proveedores` · `PATCH /proveedores/{id}` (catalogo-v1) | RF-17 · DEC-12 | `/proveedores` · `ProveedoresPage` | CP-RF17-03 |
| `GET /motivos-movimiento` (movimientos-v1) | HU-06 · HU-07 | `/movimientos` · `MovimientosPage` | Sin caso propio; se usa en CP-HU06 y CP-HU07 |
| `GET /alertas/resumen` (analitica-v1) | HU-10 | Contador del menú (`TableroPage`) | CP-HU10-03 |
| `GET /analitica/tablero` (analitica-v1) | RF-19 | `/tablero` · `TableroPage` | CP-RF19-01 |
| `GET /analitica/consumo` (analitica-v1) | RF-14 | `/tablero` · `TableroPage` | CP-RF14-01 |
| `GET /analitica/cobertura` (analitica-v1) | RF-17 | `/tablero` · `TableroPage` | CP-RF17-01 |
| `GET /analitica/exportar` (analitica-v1) | RNF-10 | `/tablero` · `TableroPage` | CP-RNF10-01 |
| `GET /analitica/kardex` (analitica-v1) | DEC-11 · A3 | Detalle del producto (X7) | CP-DEC11-01 |
| `GET /analitica/proveedores-cercanos` (analitica-v1) | RF-17 · DEC-12 | `/tablero` · `TableroPage` (mapa) | CP-RF17-02 |

## Decisiones DEC cerradas

Registro según el acta A1 (`docs/02-arquitectura/acta-A1-decisiones.md`, PR #38).

| DEC | Decisión | HU / requisitos que afecta |
|---|---|---|
| DEC-01 | Se ratifica: el rol Gerente forma parte del sistema | HU-09 · HU-08 · RF-13 · RF-14 |
| DEC-02 | Se ratifica: el Administrador tiene `usuario.gestionar` | HU-09 · RF-13 |
| DEC-03 | Se ratifica: la Manicurista solo registra salidas | HU-06 · HU-07 · RF-01 |
| DEC-04 | Se ratifica: las alertas son opcionales por producto (`stock_minimo` nulo = sin alerta) | HU-10 · RF-16 a RF-18 |
| DEC-05 | Se ratifica: el filtro por fechas es opcional | HU-08 · RF-15 |
| DEC-06 | Se ratifica: anulación controlada, sin borrado. ADR-0003 aceptado | HU-13 · RF-20 · RN-13 · RNF-06 |
| DEC-07 | p95 ≤ 1.500 ms y errores ≤ 1 % con 20 usuarios simultáneos (Q5). La disponibilidad no se mide | RNF-04 · RNF-05 |
| DEC-08 | Se ratifica: contraseña de 8 a 128 caracteres, sesión JWT de 30 minutos y recuperación por correo | HU-01 · RNF-01 |
| DEC-09 | Se ratifica: respaldo semanal cifrado, que empieza con la primera release a `main` | RNF-10 |
| DEC-10 | Se ratifica: 8 categorías y 12 unidades de medida | HU-02 · HU-04 · RF-12 |
| DEC-11 | Costo promedio ponderado, solo como valoración. El cliente es microempresa (Grupo 3) y su política está en `docs/05-contabilidad/politica-costo-inventarios.md` (A3) | HU-06 · HU-07 · RF-10 |
| DEC-12 | Proveedores georreferenciados (D12, X2, X6). Varias sedes quedan fuera del alcance | RF-17 |
