# Informe de entrega del Issue #46 — Diagnóstico arquitectónico

**Referencia de trabajo**

Refs #46

La referencia identifica el issue asociado a esta entrega. Por sí misma no constituye un pull request ni acredita el cierre del issue.

**Fuente única de resultados.** El JSON se conserva en el proyecto externo `agente-auditor-ia`, en `reports/diagnostico_arquitectura_20261123_232443_710295.json`. La carpeta documental de `spa-docs` reúne estos cinco Markdown. El informe fue producido con el motor `0.6.1.1`, esquema `1.3.0`. Los localizadores `archivo:línea` y los IDs `EV-…` se transcriben del JSON. Los localizadores identifican las fuentes observadas por el auditor.

## 1. Resumen ejecutivo

La ejecución cubrió **10 de 10 repositorios**, inspeccionó **301 archivos** y clasificó **1.531 evidencias**, de las cuales **752** integran la sección arquitectónica. Los 13 hallazgos comprobados corresponden a la comparación de matrices documentales. El único desacuerdo de títulos es HU-13.

**Recomendación de ingeniería:** mantener el monolito modular. El despliegue conjunto está documentado y el registro de salidas comparte una garantía transaccional con el stock de Catálogo. Los cuatro dominios examinados quedaron `no_evaluable`. La extracción gradual queda condicionada a nuevas evidencias y el informe no justifica una migración completa. `EV-a016ec2e893c241c8df04ef6` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:11`; `EV-080c4409b708672edfc08796` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87`.

## 2. Objetivo y alcance del Issue #46

Esta entrega documenta el diagnóstico estático ejecutado sobre los diez árboles locales y su trazabilidad documental. Usa ese diagnóstico para comparar alternativas de arquitectura y establecer qué evidencia falta para seleccionar un dominio piloto. El alcance demostrado corresponde al informe local `1.3.0`. La matriz de la sección 14 delimita el alcance de esta evidencia para el Issue #46.

## 3. Metodología y entorno de ejecución

El resultado registra un inventario local, búsqueda y clasificación de evidencias, comparación de las matrices general y detallada, y consolidación en hallazgos, entidades, relaciones e inferencias. La clasificación distingue documentación, código, configuración, plantillas y otras fuentes. Los conteos se refieren a registros y coincidencias, por lo que varias evidencias pueden pertenecer al mismo archivo.

El JSON registra `modo=solo_lectura`, motor `0.6.1.1` y localizadores bajo `/opt/auditor/repos`. La búsqueda informa redacción activa, cero errores y límite sin alcanzar. `engine_commit` y `engine_reproducible_from_commit` son nulos. El JSON deja sin acreditar la configuración efectiva del contenedor y la identidad Git exacta de las entradas.

## 4. Inventario de repositorios

| Repositorio | Disponibilidad registrada |
|---|---|
| `.github` | Disponible |
| `spa-docs` | Disponible |
| `spa-backend-api` | Disponible |
| `spa-database` | Disponible |
| `spa-frontend-web` | Disponible |
| `spa-qa` | Disponible |
| `spa-org-bootstrap` | Disponible |
| `spa-devops-workflows` | Disponible |
| `spa-infra-aws` | Disponible |
| `spa-template-servicio` | Disponible |

Los nombres y estados proceden de `inventario.repositorios`. El alcance de la inspección se cuantifica en la sección siguiente.

## 5. Arquitectura actual

- **Hecho documentado.** El backend se organiza en Identidad, Catálogo, Movimientos y Alertas y analítica. El diseño agrupa esos dominios en un despliegue conjunto como monolito modular. **S0 · Datos** se describe como plataforma administrada desde `spa-database`. `EV-a016ec2e893c241c8df04ef6` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:11`; `EV-e8afc1bb1757ddbd7b1bdc63` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:40`.
- **Hecho documentado.** El backend declara un diseño hexagonal de puertos y adaptadores. El diseño identifica una capa de dominio y adaptadores de infraestructura como SQLAlchemy y Supabase Auth. `EV-06bc25e1ff8a2f66bbd23eba` · `spa-backend-api/docs/ARQUITECTURA.md:1`; `EV-c73a1bcb2f20743a216dbcd5` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:244`; `EV-5a4839081d5ab29976ed1a4a` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:246`.
- **Observación de código.** El informe recoge una fábrica `create_app()` y la creación de una aplicación FastAPI. **Inferencia:** estos fragmentos son coherentes con el diseño documentado. La auditoría no observó la topología activa ni verificó todas las fronteras entre módulos. `EV-fe513bd255fa3621bd537ec7` · `spa-backend-api/app/main.py:8`; `EV-541a136c928d4aa6525fa361` · `spa-backend-api/app/main.py:10`.

El diagnóstico reúne cinco áreas. Los conteos siguientes corresponden a los hechos documentados y a las coincidencias candidatas del consolidado, respectivamente.

| Área | Evidencias documentales | Coincidencias candidatas | Localizador representativo |
|---|---:|---:|---|
| API backend | 48 | 25 | `EV-2f85dee1b49e14fd8bd0cf98` · `spa-backend-api/app/main.py:1` |
| Arquitectura hexagonal | 4 | 2 | `EV-06bc25e1ff8a2f66bbd23eba` · `spa-backend-api/docs/ARQUITECTURA.md:1` |
| Base de datos | 95 | 43 | `EV-e8afc1bb1757ddbd7b1bdc63` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:40` |
| Infraestructura | 106 | 68 | `EV-1987565b6a3d3fc641a8b75b` · `spa-infra-aws/k8s/base/backend/deployment-v1.yaml:26` |
| Microservicios | 3 | 1 | `EV-f902695e92cd7ceda58b5370` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:11` |

Documentos relacionados en `spa-docs`: [arquitectura de servicios](../arquitectura-servicios.md) y [matriz de trazabilidad](../../01-requisitos/TRAZABILIDAD.md). Las rutas de repositorio, las líneas y los IDs EV citados conservan los localizadores del JSON.

Documentación del auditor: [prompt](agente/prompt.md), [permisos y herramientas](agente/permisos-y-herramientas.md), [reproducibilidad](agente/reproducibilidad.md) y [protocolo de pruebas negativas](agente/prueba-negativa.md).

## 6. Resultados cuantitativos

| Indicador | Resultado |
|---|---:|
| Repositorios disponibles / esperados | 10 / 10 |
| Repositorios no disponibles | 0 |
| Archivos inspeccionados | 301 |
| Evidencias encontradas / clasificadas | 1.531 / 1.531 |
| Evidencias en el diagnóstico arquitectónico | 752 |
| Historias en matriz general / detallada | 13 / 13 |
| Endpoints transversales documentados | 13 |
| Hallazgos comprobados | 13 |
| Alertas | 1 informativa |
| Errores de búsqueda | 0 |

`estado_informe=completo` describe la ejecución registrada. `estado_evidencia=parcial` describe el alcance de las conclusiones arquitectónicas. Un número alto de coincidencias no mide rendimiento ni autonomía de los dominios.

## 7. Hallazgos documentales

**Fortaleza comprobada.** Las dos matrices contienen los mismos 13 identificadores. HU-01 a HU-12 mantienen títulos equivalentes tras normalización. Como ejemplo, Autenticación aparece en las dos filas asociadas a HU-01. `EV-66e55a8c5018fb9f628add62` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:5`; `EV-8ea3fbb55d125e3ab2f1f274` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:39`.

**Problema comprobado.** HU-13 se titula **Eliminar movimiento** en la matriz general y **Anular movimiento** en la detallada. La segunda documenta `POST /movimientos/{id}/anulaciones`. El cambio de verbo puede afectar el comportamiento esperado. Se requiere revisión humana para fijar el término y los criterios de aceptación. El informe no adjudica qué título es correcto. `EV-0910bcc9b9b0fbf265df7e9e` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:17`; `EV-4d3af130148a97ed164c520d` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:51`.

**Límite del hallazgo.** Las coincidencias, endpoints y casos citados son datos documentales. Por sí solos no acreditan que las HU estén implementadas ni que sus pruebas hayan pasado.

## 8. Comparación de alternativas arquitectónicas

Las tres figuran como `requiere_validacion` y con confianza baja. Los recuentos indican referencias asociadas a cada alternativa, sin equivaler a una puntuación de viabilidad.

| Alternativa | Resultado y evidencia representativa |
|---|---|
| Mantener el monolito modular · 408 referencias | Alternativa de referencia mientras se comprueban límites y datos. El diseño documenta una transacción que preserva la consistencia del stock. `EV-080c4409b708672edfc08796` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87`. |
| Extracción gradual · 17 referencias | Opción futura si un dominio demuestra límites, propiedad de datos y autonomía. Por ejemplo, se documenta el endpoint de tablero, cuya presencia identifica superficie funcional para evaluar. `EV-f29c4bdc5d2922c4ce2e2430` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:64`. |
| Migración completa · 305 referencias | El informe carece de evidencia suficiente para justificarla. El diseño advierte el coste de multiplicar repositorios, pipelines y despliegues, y el diagnóstico no aporta métricas de carga o rendimiento. `EV-0af9a20ba834d86ba0afa2ba` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:15`. |

## 9. Evaluación individual de los dominios

Los cuatro dominios obtuvieron **`no_evaluable`**, con confianza baja. En cada uno los seis criterios están `sin_evidencia`: cohesión, acoplamiento, propiedad de datos, consistencia transaccional, autonomía operativa y escalabilidad.

| Dominio | Superficie documental observada | Evidencia representativa |
|---|---|---|
| Identidad | Recuperación de contraseña y consulta de roles. | `EV-a9491b70ed14f1fc6563bcbb` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:57`; `EV-f3b4d903c1cea0b4bff9c944` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:58`. |
| Catálogo | Stock mínimo, administración de categorías y proveedores. | `EV-a87d31e54be9529adc9ff6e4` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:59`; `EV-8e213e9ec67a90151d0e7255` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:61`. |
| Movimientos | Entradas, salidas y consulta de motivos. | `EV-33ee7d45ae7010ec4a74fab2` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:44`; `EV-f5af2ff5cea53dd38106209f` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:45`; `EV-cf00875bb866d9f4009e3fd9` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:62`. |
| Analítica | Resumen de alertas y tablero como parte de la superficie documentada. | `EV-76223701be0974827f3c59b7` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:63`; `EV-f29c4bdc5d2922c4ce2e2430` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:64`. |

## 10. Riesgos de una migración prematura

| Dimensión | Evidencia y consecuencia condicionada |
|---|---|
| Escalabilidad y latencia | El diseño advierte el coste de las llamadas remotas y los arranques en frío. Sin mediciones de carga o latencia, una separación podría sumar viajes de red sin un beneficio demostrado. `EV-cd98eae0dae0d1a21a8f193c` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:63`. |
| Acoplamiento y consistencia | Se documentan una conexión compartida y una transacción que valida y actualiza el stock al registrar la salida. Dividir ese flujo exige demostrar que se preservan sus garantías. `EV-3f9c4bf31868792192b912c5` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:85`; `EV-080c4409b708672edfc08796` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87`. |
| Despliegue | Se observó configuración de Kubernetes y la documentación menciona manifiestos para una API y tareas pendientes. La autonomía de despliegue de cada dominio sigue sin acreditarse. `EV-1987565b6a3d3fc641a8b75b` · `spa-infra-aws/k8s/base/backend/deployment-v1.yaml:26`; `EV-cffe06dfe7a82801c0642be3` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:343`; `EV-8a384774a986cff895b25782` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:345`. |
| Complejidad operativa | La documentación prevé más repositorios, pipelines y proyectos de despliegue para un equipo de dos desarrolladores de backend. Ese coste debe contrastarse con un beneficio comprobable. `EV-0af9a20ba834d86ba0afa2ba` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:15`. |

## 11. Módulos que conviene mantener integrados

**Recomendación provisional.** Mantener **Identidad, Catálogo, Movimientos y Analítica** en la unidad de despliegue actual. Los cuatro obtuvieron `no_evaluable` y el informe no aporta una clasificación de prioridad. En particular, conservar integrados Movimientos y Catálogo evita cambiar de inmediato el límite en el que el diseño concentra la consistencia del stock. Mantener S0 como plataforma de datos documentada. `EV-a016ec2e893c241c8df04ef6` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:11`; `EV-e8afc1bb1757ddbd7b1bdc63` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:40`; `EV-080c4409b708672edfc08796` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87`.

## 12. Ruta condicionada para evaluar una extracción futura

La salida contiene un solo elemento, **`PASO-01`**, con estado **`bloqueado_por_evidencia`**. Su acción es completar la evidencia de límites, propiedad de datos y operación antes de seleccionar un piloto. No tiene dominio asignado, dependencias ni IDs de evidencia propios. La ruta representa una condición previa de evaluación. Una migración requeriría decisión posterior.

Como criterio de ingeniería, para reabrir la decisión se necesita un mapa de componentes y dependencias, responsables de datos y transacciones, y evidencia de operación y escalado independientes. El flujo de stock es un caso concreto para comprobar la consistencia antes de seleccionar un candidato. `EV-080c4409b708672edfc08796` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87`.

## 13. Limitaciones del diagnóstico

- El análisis arquitectónico trabaja sobre el consolidado de coincidencias. Una relación ausente en su grafo puede existir en el sistema.
- Los fragmentos de código, documentación, pruebas y plantillas tienen alcances distintos. Una referencia por sí sola no acredita implementación, ejecución de pruebas ni despliegue.
- No se aportaron métricas de carga, latencia o rendimiento. `cuellos_botella` y `recomendaciones` están vacíos en la salida arquitectónica. Las recomendaciones de este documento son una lectura de ingeniería, identificada como tal.
- La ejecución local no acredita una revisión Git reproducible, una importación de instantáneas GitHub ni la efectividad de permisos de GitHub.
- El estado global completo coexiste con evidencia arquitectónica parcial. La primera etiqueta se refiere a la cobertura registrada y la segunda al respaldo disponible para las conclusiones.

## 14. Matriz de cumplimiento del Issue #46

| Criterio evaluado para esta entrega | Estado | Evidencia o condición pendiente |
|---|---|---|
| Inventariar las entradas locales y producir diagnóstico `1.3.0` | **Demostrado** | `inventario`, `resumen` y `diagnostico_arquitectonico` en el JSON generado. |
| Contrastar la trazabilidad documental | **Demostrado** | 13 comprobaciones, incluida la discrepancia de HU-13 descrita en la sección 7. |
| Comparar alternativas y evaluar candidatos | **Parcial** | Se compararon tres alternativas y se documentaron cuatro evaluaciones `no_evaluable`. Falta evidencia para escoger un piloto. |
| Operar en modo de solo lectura y acreditar aislamiento efectivo | **Parcial** | El informe declara `solo_lectura`. La configuración efectiva de montajes, red y privilegios requiere evidencia operativa externa al JSON. |
| Adquisición paginada y verificación de una instantánea GitHub en el diagnóstico | **Pendiente** | Esta ejecución usa el esquema `1.3.0` y no acredita adquisición remota ni procedencia GitHub en un informe `1.4.0`. |
| Vincular issues de GitHub y cotejar árboles con commits autenticados | **Pendiente** | Las referencias documentales del JSON por sí solas no acreditan esos cotejos. |
| Verificar permisos efectivos y denegaciones de escrituras en GitHub | **Pendiente** | El JSON no contiene resultados de pruebas negativas con credenciales restringidas. |

## 15. Reproducción de la ejecución local

Esta es una receta para repetir el análisis con autorización. La fuente registra la versión, el modo, los insumos observados y el formato de salida. La configuración efectiva de infraestructura y las revisiones de los árboles deben registrarse separadamente en una futura ejecución.

1. Preparar las copias locales de los diez repositorios y conservar la misma estructura. En el entorno del auditor, su raíz es `/opt/auditor/repos`. El documento esperado es `spa-docs/docs/01-requisitos/TRAZABILIDAD.md`. Para una comparación fiel, usar las mismas entradas y registrar sus revisiones por un medio autorizado.
2. Preparar un entorno con el motor `0.6.1.1` y el esquema `1.3.0`, desde la raíz que contiene el módulo `agent`. Si se usa Docker, disponer diez montajes individuales de solo lectura y una salida separada en `/opt/auditor/reports`. Confirmar antes del arranque el aislamiento de red, la raíz de solo lectura, el usuario sin privilegios y la ausencia de credenciales.
3. Después de confirmar los controles y obtener autorización de ejecución, arrancar el entorno preparado. El comando de entrada del auditor es `python -m agent`. Conservar salida de consola y código de salida. Si el proceso falla, documentar el error antes de repetir.
4. Identificar el nuevo `diagnostico_arquitectura_*.json` en `reports/`. Comprobar que se lee, que declara `schema_version=1.3.0` y que contiene `diagnostico_arquitectonico` con `alternativas_microservicios`, `evaluacion_microservicios` y `ruta_extraccion`. Comparar cobertura, alertas y hallazgos con el informe entregado. El nombre de salida cambia con la marca temporal.

## 16. Conclusiones y recomendaciones de ingeniería

1. **Mantener el monolito modular.** El despliegue conjunto está documentado y el coste inmediato de separar dominios carece de una justificación medida. Conservar el diseño hexagonal como referencia para ordenar límites y adaptadores. `EV-a016ec2e893c241c8df04ef6` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:11`; `EV-06bc25e1ff8a2f66bbd23eba` · `spa-backend-api/docs/ARQUITECTURA.md:1`.
2. **Resolver la discrepancia HU-13 mediante revisión funcional.** Confirmar qué significa anular o eliminar y alinear el título, el contrato y los criterios de aceptación una vez acordado el comportamiento. `EV-0910bcc9b9b0fbf265df7e9e` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:17`; `EV-4d3af130148a97ed164c520d` · `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:51`.
3. **Condicionar cualquier piloto a evidencia adicional.** Comprobar el límite transaccional de stock, la propiedad de los datos y la capacidad de operar y escalar el candidato. La documentación plantea una separación futura apoyada en módulos y contratos. El informe deja su viabilidad sin acreditar. `EV-080c4409b708672edfc08796` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87`; `EV-bfc468b1db2b3926f260eb71` · `spa-docs/docs/02-arquitectura/arquitectura-servicios.md:17`.
