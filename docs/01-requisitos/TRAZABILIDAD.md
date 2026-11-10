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

Tarea A2 (spa-docs#32). Fuentes verificadas el 07-nov-2026 y estado actualizado el 10-nov-2026 (contratos y rutas en `develop`; estado según los PR de cada repositorio):

- **Endpoints:** contratos `spa-backend-api/docs/openapi/*-v1.yaml`. Cada operación declara su HU en la descripción.
- **Pantallas:** rutas de `spa-frontend-web/src/App.jsx` y `src/constants/routes.js`, y la tabla de HU de cada `src/features/*/README.md`.
- **Pruebas:** casos CP del plan de pruebas Q1 (`spa-qa/docs/PLAN_PRUEBAS.md`, sección 3). Los casos 401/403 de cada endpoint están en su sección 4.

**Estado al 10-nov-2026:**

- Backend (`develop`): con X1 (spa-backend-api#65) y X2 (spa-backend-api#66) todos los endpoints del contrato tienen implementación. El cierre formal (B12: cero 501 declarados en el contrato, cobertura ≥ 80 % y README) está en PR.
- Frontend (`develop`): todas las pantallas integradas, incluido el tablero de decisiones (F15, spa-frontend-web#42) y las correcciones de la auditoría de accesibilidad WCAG 2.2 AA (F16, spa-frontend-web#47).
- Verificación de QA en staging: suite Q2 de identidad, permisos y catálogo (spa-qa#9), prueba de carga Q5 (spa-qa#10) y pruebas de QA por PR registradas en cada issue.
- Pendientes para R2: datos de demostración (D11) y cierre del backend (B12).

| HU | Endpoint (contrato) | Pantalla (ruta · componente) | Pruebas (Q1) | Estado al 10-nov |
|---|---|---|---|---|
| HU-01 Autenticación | `POST /auth/login` · `GET /auth/me` (identidad-v1) | `/login` · `LoginPage`; menú según permisos (`MENU`) | CP-HU01-01 a CP-HU01-08 | Integrado: API B5 (spa-backend-api#45), pantalla F6 (spa-frontend-web#30) |
| HU-02 Registrar producto | `POST /productos` · `GET /unidades-medida` (catalogo-v1) | `/inventario` · `InventarioPage` (formulario de producto) | CP-HU02-01 a CP-HU02-06 | Integrado: API B9 (spa-backend-api#54), pantalla spa-frontend-web#25 |
| HU-03 Consultar inventario | `GET /productos` · `GET /productos/{id}` (catalogo-v1) | `/inventario` · `InventarioPage` (tabla con filtros) | CP-HU03-01 a CP-HU03-07 | Integrado: API B8 (spa-backend-api#43), pantalla spa-frontend-web#25 |
| HU-04 Filtrar por categoría | `GET /categorias` · `GET /productos?categoria_id=` (catalogo-v1) | `/inventario` · `InventarioPage` (filtro de categoría) | CP-HU04-01, CP-HU04-02 | Integrado: B8 (spa-backend-api#43), `GET /categorias` con B9 (spa-backend-api#54) y verificado en staging; pantalla spa-frontend-web#25 |
| HU-05 Actualizar producto | `PATCH /productos/{id}` con `If-Match` (catalogo-v1) | `/inventario` · `InventarioPage` (edición con `version`) | CP-HU05-01 a CP-HU05-03 | Integrado: API B9 (spa-backend-api#54), pantalla spa-frontend-web#25 |
| HU-06 Registrar entrada | `POST /movimientos/entradas` (movimientos-v1) | `/movimientos` · `MovimientosPage` | CP-HU06-01 a CP-HU06-04 | Integrado: B10 (spa-backend-api#62), F12 (spa-frontend-web#45), motivos X1 (spa-backend-api#65, spa-frontend-web#44) |
| HU-07 Registrar salida | `POST /movimientos/salidas` (movimientos-v1) | `/movimientos` · `MovimientosPage` | CP-HU07-01 a CP-HU07-03 | Integrado: B10 (spa-backend-api#62), F12 (spa-frontend-web#45), motivos X1 (spa-backend-api#65, spa-frontend-web#44) |
| HU-08 Consultar historial | `GET /movimientos` · `GET /movimientos/{id}` (movimientos-v1) | `/historial` · `HistorialPage` | CP-HU08-01 a CP-HU08-03 | Integrado: B11 (spa-backend-api#63), F13 (spa-frontend-web#46) |
| HU-09 Gestionar usuarios | `GET`/`POST /usuarios` · `GET`/`PATCH /usuarios/{id}` · `PATCH /usuarios/{id}/estado` (identidad-v1) | `/usuarios` · `UsuariosPage` | CP-HU09-01 a CP-HU09-07 | Integrado: B6 (spa-backend-api#52), pantalla F8 en develop |
| HU-10 Alertas de bajo stock | `GET /alertas` (analitica-v1) | `/tablero` · `TableroPage` (lista de alertas) | CP-HU10-01 a CP-HU10-03 | Integrado: pantalla X4 (spa-frontend-web#34) |
| HU-11 Productos agotados | `POST /movimientos/salidas` · `GET /alertas` | `/movimientos` y `/tablero` | CP-HU11-01 | Integrado: B10 (spa-backend-api#62), X4 (spa-frontend-web#34) |
| HU-12 Descontinuar producto | `POST /productos/{id}/descontinuar` · `POST /productos/{id}/reactivar` (catalogo-v1) | `/inventario` · `InventarioPage` | CP-HU12-01 a CP-HU12-03 | Integrado: API B9 (spa-backend-api#54), pantalla spa-frontend-web#25 |
| HU-13 Anular movimiento | `POST /movimientos/{id}/anulaciones` (movimientos-v1). No existe `DELETE` (DEC-06) | `/historial` · `HistorialPage` (anulación con confirmación) | CP-HU13-01 a CP-HU13-03 | Integrado: B11 (spa-backend-api#63), F13 (spa-frontend-web#46) |

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

### Decisiones de PO posteriores al acta A1

| Fecha | Decisión | HU / requisitos que afecta |
|---|---|---|
| 09-nov-2026 | El mapa de proveedores parte de la sede del spa (Calle 57 # 7-11, Bogotá) con su marcador propio y solo lo ven Administrador y Gerente (permiso `analitica.consultar`); la Manicurista no lo necesita. La vista de varias sedes queda como escenario de escalabilidad (spa-frontend-web#41) | RF-17 · DEC-12 |
