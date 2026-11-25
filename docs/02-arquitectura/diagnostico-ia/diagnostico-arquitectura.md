# Informe de entrega del Issue #46 — Diagnóstico arquitectónico de SPA-UNAS

Refs #46

Documento de revisión para el PR #47. Integra los resultados históricos del auditor contenedorizado, el análisis arquitectónico con LLM sobre commits locales fijados y la verificación independiente de cinco grupos de evidencia. La referencia al Issue #46 identifica el alcance de la entrega. Su cierre depende de las verificaciones enumeradas al final.

El análisis detallado de dominios, riesgos, seguridad y alternativas se presenta en el [anexo técnico LLM](./analisis-tecnico-llm.md).

## 1. Resumen ejecutivo

Resultado histórico del auditor contenedorizado. La ejecución cubrió 10 de 10 repositorios, inspeccionó 301 archivos y clasificó 1.531 evidencias, de las cuales 752 se incluyeron en el diagnóstico arquitectónico. Los 13 hallazgos comprobados corresponden a la comparación de matrices documentales. El único desacuerdo de títulos observado fue HU-13. El motor clasificó los cuatro dominios como no_evaluable para extracción, con confianza baja.

Decisión recomendada para el PR #47. Conservar el monolito modular y mantener Catálogo y Movimientos bajo una autoridad transaccional común. El código agrupa cuatro dominios en una aplicación FastAPI. PostgreSQL bloquea la fila del producto al aplicar un movimiento y actualiza saldo y valoración en la misma transacción. Separar procesos puede preservar esa atomicidad si la operación completa permanece en una única transacción PostgreSQL. En ese escenario persiste el acoplamiento de datos. spa-backend-api@2a0a2d41431f:app/main.py:12–18,41–45 · spa-backend-api@2a0a2d41431f:app/modulos/catalogo/infrastructure/repositorio_sqlalchemy.py:127–189 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:100–106,133–198.

Evolución condicionada. Analítica, empezando por la exportación, es candidata a un piloto si las mediciones muestran que el consumo de memoria, la duración de la solicitud o la carga SQL degradan el tráfico transaccional y las optimizaciones internas resultan insuficientes. El código acredita la materialización del archivo durante la solicitud. Q5 aporta una línea base histórica de consulta y salida de inventario, con alcance limitado a ese escenario. spa-backend-api@2a0a2d41431f:app/modulos/analitica/infrastructure/exportador_duckdb.py:116–141 · spa-backend-api@2a0a2d41431f:app/modulos/analitica/api/router.py:164–196 · spa-qa@9be19075107a5:suites/carga/reporte-q5-2026-11-09.md:3–20.

## 2. Alcance y lectura de la evidencia

| Fuente | Qué acredita | Límite |
|---|---|---|
| Auditoría estática contenedorizada | Inventario, coincidencias, resultados estructurados y localizadores EV-… del informe histórico 1.3.0. | El JSON carece del commit del motor y de los commits exactos de las entradas. Sus recuentos conservan el alcance de esa ejecución. |
| Análisis LLM sobre objetos Git | Contenido de diez revisiones locales fijadas, citado como repositorio@commit:ruta:línea. | Los commits identifican los árboles analizados en esta revisión. La vigencia de las referencias en GitHub y la correspondencia con despliegues activos quedan sin verificar. |
| Verificación independiente | Cotejo directo de inventario, Analítica, ADR, Q5 y seguridad. | Fue una segunda lectura estática. Las pruebas, migraciones, despliegues y denegaciones remotas del PAT quedaron fuera de la ejecución. |

Hecho comprobado identifica contenido observado en una fuente. Inferencia identifica una consecuencia técnica derivada de ese contenido. Hipótesis señala un riesgo o beneficio que requiere medición. No evaluable indica que falta evidencia suficiente. Implementado se aplica a código, SQL, pruebas o configuración versionados. Documentado se aplica a descripciones, decisiones y resultados reportados. Propuesto se aplica al trabajo planteado.

## 3. Auditoría estática contenedorizada

### 3.1. Identidad, metodología y seguridad de la corrida

La fuente de los resultados históricos es reports/diagnostico_arquitectura_20261123_232443_710295.json, conservado en el proyecto externo agente-auditor-ia. El informe se produjo con el motor 0.6.1.1, esquema 1.3.0, y registra modo=solo_lectura. Los IDs EV-… y los localizadores se conservan como aparecen en el diagnóstico original. El directorio /opt/auditor/repos corresponde al entorno de esa corrida.

El procedimiento registró inventario local, búsqueda y clasificación de evidencias, comparación de las matrices general y detallada, y consolidación en hallazgos, entidades, relaciones e inferencias. La clasificación distingue documentación, código, configuración, plantillas y otras fuentes. Varias evidencias pueden pertenecer a un mismo archivo. La búsqueda informó redacción activa, cero errores y límite sin alcanzar. engine_commit y engine_reproducible_from_commit quedaron nulos.

El registro complementario de inspección del contenedor identifica la imagen agente-auditor-ia:0.6.1.1, plataforma linux/amd64, diez montajes de repositorios de solo lectura y un montaje escribible separado para informes. Documenta raíz de solo lectura, red deshabilitada, usuario sin privilegios y código de salida 0. Estos datos están atribuidos al contenedor inspeccionado. El JSON por sí solo deja sin acreditar los ajustes efectivos del contenedor y los SHA de entrada. La documentación del auditor tampoco acredita un prompt histórico de lenguaje natural. python -m agent queda registrado como comando de entrada. [Permisos y herramientas](./agente/permisos-y-herramientas.md) · [Reproducibilidad del auditor](./agente/reproducibilidad.md) · [Prompt y ejecución](./agente/prompt.md).

### 3.2. Inventario registrado

| Repositorio | Disponibilidad registrada |
|---|---|
| .github | Disponible |
| spa-docs | Disponible |
| spa-backend-api | Disponible |
| spa-database | Disponible |
| spa-frontend-web | Disponible |
| spa-qa | Disponible |
| spa-org-bootstrap | Disponible |
| spa-devops-workflows | Disponible |
| spa-infra-aws | Disponible |
| spa-template-servicio | Disponible |

Los nombres y estados proceden de inventario.repositorios.

### 3.3. Arquitectura observada por el auditor

- Hecho documentado. El diseño agrupa Identidad, Catálogo, Movimientos y Alertas y analítica en un despliegue conjunto como monolito modular. S0 Datos se describe como plataforma administrada desde spa-database. EV-a016ec2e893c241c8df04ef6 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:11 · EV-e8afc1bb1757ddbd7b1bdc63 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:40.
- Hecho documentado. El backend declara un diseño hexagonal de puertos y adaptadores, con SQLAlchemy y Supabase Auth entre sus adaptadores. EV-06bc25e1ff8a2f66bbd23eba · spa-backend-api/docs/ARQUITECTURA.md:1 · EV-c73a1bcb2f20743a216dbcd5 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:244 · EV-5a4839081d5ab29976ed1a4a · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:246.
- Observación de código e inferencia histórica. El auditor recogió la fábrica create_app() y la creación de una aplicación FastAPI, fragmentos coherentes con el diseño. La topología activa y el conjunto de fronteras entre módulos quedaron fuera de lo acreditado por esa corrida. EV-fe513bd255fa3621bd537ec7 · spa-backend-api/app/main.py:8 · EV-541a136c928d4aa6525fa361 · spa-backend-api/app/main.py:10.

| Área | Evidencias documentales | Coincidencias candidatas | Localizador representativo |
|---|---:|---:|---|
| API backend | 48 | 25 | EV-2f85dee1b49e14fd8bd0cf98 · spa-backend-api/app/main.py:1 |
| Arquitectura hexagonal | 4 | 2 | EV-06bc25e1ff8a2f66bbd23eba · spa-backend-api/docs/ARQUITECTURA.md:1 |
| Base de datos | 95 | 43 | EV-e8afc1bb1757ddbd7b1bdc63 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:40 |
| Infraestructura | 106 | 68 | EV-1987565b6a3d3fc641a8b75b · spa-infra-aws/k8s/base/backend/deployment-v1.yaml:26 |
| Microservicios | 3 | 1 | EV-f902695e92cd7ceda58b5370 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:11 |

Los localizadores anteriores pertenecen al JSON histórico. Las rutas homónimas en la revisión LLM corresponden a los commits de la sección 7. [Arquitectura de servicios](../arquitectura-servicios.md) · [Matriz de trazabilidad](../../01-requisitos/TRAZABILIDAD.md).

### 3.4. Resultados cuantitativos

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

estado_informe=completo describe la cobertura de la ejecución registrada. estado_evidencia=parcial describe el alcance de sus conclusiones arquitectónicas. Los recuentos cuantifican coincidencias y registros del auditor.

### 3.5. Hallazgos documentales

- Coincidencia comprobada. Las matrices contienen los mismos 13 identificadores. HU-01 a HU-12 mantienen títulos equivalentes tras normalización. Autenticación es el ejemplo documentado en ambas filas de HU-01. EV-66e55a8c5018fb9f628add62 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:5 · EV-8ea3fbb55d125e3ab2f1f274 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:39.
- Discrepancia comprobada. HU-13 figura como «Eliminar movimiento» en la matriz general y «Anular movimiento» en la detallada. La segunda documenta POST /movimientos/{id}/anulaciones. El motor señaló una diferencia con impacto potencial en los criterios de aceptación y dejó su resolución a revisión humana. EV-0910bcc9b9b0fbf265df7e9e · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:17 · EV-4d3af130148a97ed164c520d · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:51.

Límite del hallazgo. La comparación de matrices solo acredita coincidencias y discrepancias documentales. La implementación de cada historia de usuario y los resultados de sus pruebas requieren evidencia propia.

El hallazgo histórico acredita la diferencia documental. La sección 5 registra el cotejo posterior de la anulación contra DEC-06 y el código fijado.

### 3.6. Alternativas y evaluación histórica de dominios

Las tres alternativas figuran como requiere_validacion, con confianza baja. Las cifras de 408, 17 y 305 cuentan referencias asociadas por el motor a cada alternativa. La viabilidad arquitectónica se evalúa con criterios técnicos y evidencia operativa.

| Alternativa | Resultado histórico y evidencia representativa |
|---|---|
| Mantener el monolito modular · 408 referencias | Alternativa de referencia mientras se comprueban límites y datos. El diseño documenta la transacción de stock. EV-080c4409b708672edfc08796 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87. |
| Extracción gradual · 17 referencias | Opción futura si un dominio demuestra límites, propiedad de datos y autonomía. El endpoint de tablero es una superficie funcional documentada para evaluar. EV-f29c4bdc5d2922c4ce2e2430 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:64. |
| Migración completa · 305 referencias | El consolidado histórico carece de evidencia suficiente para justificarla y el diseño advierte el costo de multiplicar repositorios, pipelines y despliegues. EV-0af9a20ba834d86ba0afa2ba · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:15. |

Los cuatro dominios obtuvieron no_evaluable con confianza baja. Los seis criterios del motor quedaron sin_evidencia en cada dominio: cohesión, acoplamiento, propiedad de datos, consistencia transaccional, autonomía operativa y escalabilidad.

| Dominio | Superficie documental observada | Evidencia representativa |
|---|---|---|
| Identidad | Recuperación de contraseña y consulta de roles. | EV-a9491b70ed14f1fc6563bcbb · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:57 · EV-f3b4d903c1cea0b4bff9c944 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:58 |
| Catálogo | Stock mínimo, administración de categorías y proveedores. | EV-a87d31e54be9529adc9ff6e4 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:59 · EV-8e213e9ec67a90151d0e7255 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:61 |
| Movimientos | Entradas, salidas y consulta de motivos. | EV-33ee7d45ae7010ec4a74fab2 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:44 · EV-f5af2ff5cea53dd38106209f · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:45 · EV-cf00875bb866d9f4009e3fd9 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:62 |
| Analítica | Resumen de alertas y tablero. | EV-76223701be0974827f3c59b7 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:63 · EV-f29c4bdc5d2922c4ce2e2430 · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:64 |

### 3.7. Riesgos, ruta y recomendaciones históricas

| Dimensión | Evidencia y consecuencia condicionada |
|---|---|
| Escalabilidad y latencia | El diseño advierte el costo de llamadas remotas y arranques en frío. Sin mediciones dentro del consolidado histórico, el beneficio de una separación quedó por demostrar. EV-cd98eae0dae0d1a21a8f193c · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:63. |
| Acoplamiento y consistencia | Se documentan una conexión compartida y una transacción que valida y actualiza stock al registrar una salida. Dividir ese flujo requiere demostrar que se preservan sus garantías. EV-3f9c4bf31868792192b912c5 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:85 · EV-080c4409b708672edfc08796 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87. |
| Despliegue | Se observaron manifiestos de Kubernetes para una API y tareas pendientes. La autonomía de despliegue de los dominios quedó sin acreditar. EV-1987565b6a3d3fc641a8b75b · spa-infra-aws/k8s/base/backend/deployment-v1.yaml:26 · EV-cffe06dfe7a82801c0642be3 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:343 · EV-8a384774a986cff895b25782 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:345. |
| Complejidad operativa | El diseño prevé más repositorios, pipelines y despliegues para un equipo de dos desarrolladores de backend. Ese costo requiere contrastarse con un beneficio comprobable. EV-0af9a20ba834d86ba0afa2ba · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:15. |

La salida contiene un solo elemento de ruta, PASO-01, con estado bloqueado_por_evidencia. Solicita completar la evidencia de límites, propiedad de datos y operación antes de seleccionar un piloto. Carece de dominio asignado, dependencias e IDs de evidencia propios.

El criterio de ingeniería histórico para reabrir la decisión asociada a PASO-01 requería un mapa de componentes y dependencias, la identificación de responsables de datos y transacciones, y evidencia de operación y escalado independientes. El flujo de stock era el caso concreto para comprobar la consistencia antes de seleccionar un candidato. EV-080c4409b708672edfc08796 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87.

La recomendación provisional de ingeniería fue mantener juntos los cuatro dominios, preservar S0 como plataforma de datos y conservar el diseño hexagonal. También se propuso revisar HU-13 y condicionar cualquier piloto a comprobar el límite transaccional, la propiedad de datos, la operación y el escalado. El motor no asignó una prioridad a los dominios. EV-a016ec2e893c241c8df04ef6 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:11 · EV-e8afc1bb1757ddbd7b1bdc63 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:40 · EV-06bc25e1ff8a2f66bbd23eba · spa-backend-api/docs/ARQUITECTURA.md:1 · EV-0910bcc9b9b0fbf265df7e9e · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:17 · EV-4d3af130148a97ed164c520d · /opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md:51 · EV-080c4409b708672edfc08796 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:87 · EV-bfc468b1db2b3926f260eb71 · spa-docs/docs/02-arquitectura/arquitectura-servicios.md:17.

### 3.8. Límites de la corrida histórica

El motor trabajó sobre un consolidado de coincidencias. Una relación ausente del grafo puede existir en el sistema. Los fragmentos documentales, de código, configuración, prueba o plantilla tienen alcances distintos. El consolidado 1.3.0 no aportó métricas de carga, latencia o rendimiento y dejó vacíos cuellos_botella y recomendaciones. Las recomendaciones históricas de este documento procedían de una lectura de ingeniería. Q5 entra en esta entrega por el cotejo Git posterior, documentado en la sección 5.

La corrida histórica tampoco acredita una adquisición de instantáneas GitHub, las revisiones Git exactas de sus propias entradas ni la efectividad de permisos remotos. Su cobertura completa coexiste con evidencia arquitectónica parcial.

## 4. Análisis LLM sobre commits fijados

Hecho comprobado · Implementado. El backend registra Identidad, Catálogo, Movimientos y Analítica en una aplicación FastAPI. Identidad integra Supabase Auth y aporta el principal y los permisos que consumen las rutas. Catálogo puede registrar un saldo inicial en la misma sesión en que crea el producto. Movimientos aplica el cambio de saldo y costo en PostgreSQL. Analítica lee vistas de inventario y movimientos y materializa las exportaciones durante la solicitud. El [anexo técnico LLM](./analisis-tecnico-llm.md) desarrolla las dependencias por dominio, las historias asociadas y los riesgos a medir. spa-backend-api@2a0a2d41431f:app/main.py:12–18,41–45 · spa-backend-api@2a0a2d41431f:app/compartido/seguridad.py:38–88 · spa-backend-api@2a0a2d41431f:app/modulos/identidad/infrastructure/supabase_auth.py:62–95 · spa-backend-api@2a0a2d41431f:app/modulos/catalogo/infrastructure/repositorio_sqlalchemy.py:127–189 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:100–106,177–198 · spa-backend-api@2a0a2d41431f:app/modulos/analitica/infrastructure/consultas_sql.py:54–181 · spa-backend-api@2a0a2d41431f:app/modulos/analitica/infrastructure/exportador_duckdb.py:116–141.

| Tema | Resultado de la revisión Git |
|---|---|
| ADR-0009 | Documentado. La sección 1 de arquitectura de servicios registra la decisión, sin archivo independiente ni etiqueta formal de estado en el árbol fijado. Prevé una demostración en EKS de la API conjunta y deja la extracción fuera del plan de entregas. El punto de entrada y el README del backend aún mencionan separación en Sprint 4. spa-docs@aad24346c1cc:docs/02-arquitectura/arquitectura-servicios.md:9–20,341–354 · spa-backend-api@2a0a2d41431f:app/main.py:1–4 · spa-backend-api@2a0a2d41431f:README.md:170–172. |
| ADR-0010 | Propuesto. Describe broker PostgreSQL, cola, outbox, reintentos y ejecución programada. La revisión de los árboles fijados no localizó el módulo, contrato, workflow ni migración de la cola descritos. El trigger de inventario inspeccionado actualiza el producto y devuelve el movimiento. spa-docs@aad24346c1cc:docs/02-arquitectura/adr/0010-broker-de-trabajos-priorizado.md:1–6,22–41,54–61 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:177–198. |
| Vercel, AWS y EKS | Hay configuración de Vercel para la SPA y documentación para la API. ECR está declarado en Terraform. Kubernetes e Istio tienen manifiestos para la API y su canario. EKS figura en la hoja de ruta y como módulo comentado. El aprovisionamiento y los despliegues activos quedan no evaluables. spa-frontend-web@31ea474bd407:vercel.json:1–17 · spa-docs@aad24346c1cc:docs/02-arquitectura/arquitectura-servicios.md:324–345 · spa-infra-aws@e92e218c0518:terraform/envs/staging/main.tf:19–34 · spa-infra-aws@e92e218c0518:k8s/istio/virtual-service-canary.yaml:9–14. |
| Rendimiento | Hipótesis a medir. Una base compartida, la espera de conexiones o bloqueos y la materialización de exportaciones pueden afectar el rendimiento bajo carga mixta. El código y la configuración identifican qué instrumentar. El beneficio de una extracción permanece sin demostrar. spa-backend-api@2a0a2d41431f:app/compartido/db.py:19–44 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:100–106 · spa-backend-api@2a0a2d41431f:app/modulos/analitica/infrastructure/exportador_duckdb.py:116–141. |
| Seguridad y DevSecOps | JWT, comprobaciones de permisos, RLS, roles de servicio y workflows son controles versionados. La identidad efectiva de PostgreSQL, el contexto que recibe auth.uid(), la aplicación de migraciones, las reglas de rama y los resultados de CI/CD requieren evidencia operativa. spa-backend-api@2a0a2d41431f:app/compartido/seguridad.py:38–88 · spa-backend-api@2a0a2d41431f:app/compartido/db.py:19–44 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:224–251 · spa-database@971d27ca9059:supabase/migrations/20261109100000_roles_servicios.sql:17–36,247–292 · spa-backend-api@2a0a2d41431f:.github/workflows/ci.yml:16–33. |

## 5. Verificación independiente y correcciones

| Grupo | Resultado del cotejo |
|---|---|
| Inventario | Confirmada la transacción que aplica movimiento, stock y valoración. Parcialmente confirmada la afirmación de que separar Catálogo y Movimientos exige rediseñarla. El rediseño es necesario al dividir la autoridad de escritura o cruzar la frontera de transacción. La cita de la protección frente a truncado se amplió a las líneas que declaran y ejecutan el trigger. spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:100–106,133–198,218–222 · spa-backend-api@2a0a2d41431f:app/modulos/catalogo/infrastructure/repositorio_sqlalchemy.py:127–189. |
| Analítica | Confirmada la materialización durante la solicitud. El exportador reúne todas las filas, construye un DataFrame, genera un archivo temporal y lee su contenido completo. No demostrado el beneficio operativo de extraer. spa-backend-api@2a0a2d41431f:app/modulos/analitica/infrastructure/exportador_duckdb.py:116–141 · spa-backend-api@2a0a2d41431f:app/modulos/analitica/api/router.py:164–196. |
| ADR | Confirmados el registro de ADR-0009, la discrepancia sobre Sprint 4 y el estado propuesto de ADR-0010. El cotejo de los árboles fijados no encontró los artefactos del broker descritos en la propuesta. spa-docs@aad24346c1cc:docs/02-arquitectura/arquitectura-servicios.md:9–20,341–348 · spa-backend-api@2a0a2d41431f:app/main.py:1–4 · spa-docs@aad24346c1cc:docs/02-arquitectura/adr/0010-broker-de-trabajos-priorizado.md:1–6,22–41. |
| Q5 | Confirmados los valores históricos. Refutada la afirmación del primer informe LLM de que el JSON bruto faltaba del árbol. q5-summary.json existe en la raíz de spa-qa. El enlace relativo del reporte lo busca dentro de suites/carga/. Desde allí, la ruta al archivo versionado es ../../q5-summary.json. spa-qa@9be19075107a5:suites/carga/reporte-q5-2026-11-09.md:3–20 · spa-qa@9be19075107a5:q5-summary.json:25–40,123–151,162–171. |
| Seguridad | Confirmada la definición versionada de la conexión, RLS y roles. No demostrada su eficacia en ejecución. El adaptador usa la credencial configurada tanto para login como para el alta administrativa. Se amplió el rango de recuperación web para incluir la solicitud con clave publicable y token de recuperación. spa-backend-api@2a0a2d41431f:app/compartido/db.py:19–44 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:61–65,224–251 · spa-database@971d27ca9059:supabase/migrations/20261109100000_roles_servicios.sql:17–36,247–292 · spa-backend-api@2a0a2d41431f:app/modulos/identidad/infrastructure/supabase_auth.py:62–74,89–95 · spa-frontend-web@31ea474bd407:src/api/supabase.js:20–28. |

### Q5 · Resultados históricos cotejados

El script y el reporte describen 20 usuarios virtuales y una iteración por usuario. El resumen bruto confirma 20 iteraciones, 20 consultas y 20 salidas, y coincide con las cifras redondeadas del reporte. La corrida del 9 de noviembre de 2026 cumplió DEC-07 para el escenario descrito. spa-qa@9be19075107a5:suites/carga/q5_inventario_salidas.js:16–30 · spa-qa@9be19075107a5:suites/carga/reporte-q5-2026-11-09.md:3–20 · spa-qa@9be19075107a5:q5-summary.json:25–40,54–76,123–151,162–171.

| Indicador | Resultado histórico | Umbral reportado |
|---|---:|---:|
| p95 HTTP global | 623,87 ms | ≤ 1.500 ms |
| p95 de consulta de inventario | 304,13 ms | ≤ 1.500 ms |
| p95 de registro de salida | 626,46 ms | ≤ 1.500 ms |
| Errores HTTP | 0 % (0/41) | ≤ 1 % |
| Errores del escenario | 0 % (0/40) | ≤ 1 % |
| Máximo de usuarios virtuales activos | 20 | 20 |

No se reprodujo la prueba ni se identificaron los commits desplegados durante esa corrida. El escenario ejercita login, consulta de producto y salida de inventario. La capacidad sostenida, la interferencia de exportaciones y la saturación del pooler quedan fuera de lo acreditado por Q5. spa-qa@9be19075107a5:suites/carga/q5_inventario_salidas.js:16–30,34–50,53–92 · spa-qa@9be19075107a5:suites/carga/reporte-q5-2026-11-09.md:3–20.

### HU-13 · Cotejo posterior de la regla funcional

El hallazgo histórico se conserva. En el commit local fijado, la matriz general mantiene «Eliminar movimiento» y la detallada usa «Anular movimiento». DEC-06 y ADR-0003 ratifican la anulación controlada, y el backend versiona la ruta POST correspondiente. Según el contexto de la entrega, QA informó una corrección en el PR #48. La comprobación local acredita la discrepancia de la instantánea inspeccionada. El estado actual del PR #48 y de GitHub queda fuera de esta revisión. spa-docs@aad24346c1cc:docs/01-requisitos/TRAZABILIDAD.md:17,51,82 · spa-docs@aad24346c1cc:docs/02-arquitectura/adr/0003-anulacion-de-movimientos.md:1–15 · spa-backend-api@2a0a2d41431f:app/modulos/movimientos/api/router.py:196–217.

## 6. Decisión y plan de evolución

| Alternativa | Decisión para esta entrega |
|---|---|
| Monolito modular optimizado | Mantener como línea base. Conserva la composición de la API y la autoridad transaccional común para inventario. Medir carga sostenida, contención y efecto de Analítica. spa-backend-api@2a0a2d41431f:app/main.py:41–45 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:100–106,133–198. |
| Extracción parcial | Considerar mediante piloto condicionado. Empezar por exportación solo si las mediciones identifican un límite y las optimizaciones internas resultan insuficientes. Si se conserva la misma base, medir por separado el aislamiento de cómputo y la competencia SQL. spa-backend-api@2a0a2d41431f:app/modulos/analitica/infrastructure/exportador_duckdb.py:39–46,67–74,116–141. |
| Microservicios por dominio | Diferir. Requieren beneficio medido, contratos compatibles, autoridad clara de datos, preservación de invariantes, responsables operativos y reversión demostrada. spa-docs@aad24346c1cc:docs/02-arquitectura/arquitectura-servicios.md:13–20,347–354 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:24–45,100–106,177–198. |

| Prioridad | Responsables | Resultado esperado |
|---|---|---|
| P0 · Trazabilidad y decisión | Arquitectura, PO y QA | Conciliar las referencias a Sprint 4 y EKS. Cotejar la revisión que alinee HU-13 con DEC-06. Conservar citas por SHA y alcance de la evidencia. |
| P0 · Controles efectivos | Datos, Backend y DevSecOps | Acreditar migraciones, rol de conexión, membresías, propietario de funciones y contexto de usuario. Conservar pruebas autorizadas de operaciones permitidas y denegadas. |
| P1 · Línea base y entrega reproducible | QA, Backend, Datos y Plataforma | Repetir Q5 con SHA desplegados, ampliar a carga mixta y sostenida, y verificar saldo y valoración. Medir conexiones, SQL, locks y memoria. Vincular pipelines, resultados, artefactos y entornos. |
| P2 · Pilotos condicionados | Arquitectura, PO, Backend, Datos, Plataforma y QA | Para exportación, exigir mejora reproducible frente al monolito optimizado, equivalencia funcional y reversión. Para outbox, exigir requisito aprobado, atomicidad, recuperación, idempotencia del efecto y plazo de entrega medido. |

Los criterios de aceptación por entorno y candidato se desarrollan en el [plan incremental del anexo técnico](./analisis-tecnico-llm.md#10-plan-incremental-y-conclusión). La selección de un piloto sigue condicionada a esas evidencias.

## 7. Reproducibilidad

### 7.1. Corrida contenedorizada de referencia

La receta histórica corresponde al motor 0.6.1.1, esquema 1.3.0, Docker Desktop 29.7.2, motor Linux amd64 y Python 3.12. La documentación registra la imagen agente-auditor-ia:0.6.1.1, el ID sha256:c519aab85afd19d3ff0384cfca81c81a1e46c5ee92fad032c518a5e6f78a66ca, estado final exited, código 0 y el JSON reports/diagnostico_arquitectura_20261123_232443_710295.json. Estos datos identifican la corrida histórica descrita en [reproducibilidad del auditor](./agente/reproducibilidad.md).

La siguiente receta se conserva para una repetición autorizada. Esta revisión Git no volvió a ejecutarla.

1. Preparar las copias locales de los diez repositorios y conservar la misma estructura. En el entorno del auditor, su raíz es /opt/auditor/repos. El documento esperado es spa-docs/docs/01-requisitos/TRAZABILIDAD.md. Para una comparación fiel, usar las mismas entradas y registrar sus revisiones por un medio autorizado.
2. Preparar el entorno con el motor 0.6.1.1 y el esquema 1.3.0, desde la raíz que contiene el módulo agent. Si se usa Docker, disponer diez montajes individuales de solo lectura y una salida separada en /opt/auditor/reports. Confirmar antes del arranque el aislamiento de red, la raíz de solo lectura, el usuario sin privilegios y la ausencia de credenciales.
3. Después de confirmar los controles y obtener autorización de ejecución, arrancar el entorno preparado. El comando de entrada del auditor es python -m agent. Conservar salida de consola y código de salida. Si el proceso falla, documentar el error antes de repetir.
4. Identificar el nuevo diagnostico_arquitectura_*.json en reports/. Comprobar que se lee, que declara schema_version=1.3.0 y que contiene diagnostico_arquitectonico con alternativas_microservicios, evaluacion_microservicios y ruta_extraccion. Comparar cobertura, alertas y hallazgos con el informe entregado. El nombre de salida cambia con la marca temporal.

La receta detallada conserva los comandos Docker, la verificación de la imagen local, las opciones de aislamiento, la inspección de montajes y las condiciones previas al arranque. La reproducción fiel de la corrida también depende de recuperar sus entradas exactas. [Receta Docker](./agente/reproducibilidad.md) · [Permisos y herramientas](./agente/permisos-y-herramientas.md) · [Prompt y ejecución](./agente/prompt.md).

### 7.2. Revisión LLM sobre objetos Git

Datos de sesión informados para la entrega. Modelo GPT-6.2-Sol, cliente Codex CLI v0.171.0-alpha.13 y modo Read Only (never). Se enumeraron los árboles mediante git ls-tree y se leyeron archivos mediante git show <commit>:<ruta>, con búsquedas sobre objetos locales y cotejo de los rangos citados. La verificación independiente volvió a consultar fuentes pertinentes de los mismos commits. No se usaron red ni credenciales y no se realizaron escrituras, cambios de rama, pruebas o despliegues.

| Repositorio | Referencia local usada para fijar la revisión | SHA completo utilizado |
|---|---|---|
| .github | HEAD local | be19ef416c60ebfb92e3ab8e82cdd7e4ff300b18 |
| spa-backend-api | refs/remotes/origin/develop local | 2a0a2d41431f397dbeced31243c6b1863499139b |
| spa-database | refs/remotes/origin/develop local | 971d27ca9059adca348167714839715599bc849d |
| spa-devops-workflows | HEAD local | 0cd3a1361909b8a097cb4ca8be5df82d6fedae06 |
| spa-docs | HEAD local | aad24346c1ccc575ccac54dd7f4b188a10574660 |
| spa-frontend-web | refs/remotes/origin/develop local | 31ea474bd4075b78190ce0cc84917fe8eadb2bec |
| spa-infra-aws | refs/remotes/origin/develop local | e92e218c0518adcb355ff6538e6c46378b7af475 |
| spa-org-bootstrap | HEAD local | 15544b74245c11314be9ef45fc41d5a5b4af8a03 |
| spa-qa | refs/remotes/origin/develop local | 9be19075107a5dad05c72b0311f7df0682ffddfb |
| spa-template-servicio | HEAD local | 52870ed2ccccc516f73e16bb03cb9392cb75b836 |

**Cierre de la revisión con Codex CLI.** La sesión documentada empleó Codex CLI `v0.171.0-alpha.13` y reportó el modelo `GPT-6.2-Sol`. El agente examinó los diez repositorios mediante consultas de lectura a objetos Git y contrastó código, historias de usuario, ADR y resultados de QA. Su conclusión fue conservar el monolito modular, mantener la autoridad transaccional conjunta de Catálogo y Movimientos y condicionar un eventual piloto de exportación de Analítica a mediciones reproducibles. La revisión también verificó la existencia de `spa-qa/q5-summary.json`, corrigiendo la afirmación previa sobre su ausencia.

**Alcance de seguridad y reproducción.** En el registro examinado se observaron operaciones de lectura y no se observaron modificaciones ni publicaciones. El modo de solo lectura y la política `never` fueron declarados para la sesión, pero no se ejecutó una prueba independiente de rechazo de escritura ni se acreditó aislamiento de red de Codex. Los controles Docker corresponden al auditor Python, no a esta sesión LLM. El [prompt y su contexto](./agente/prompt.md), los [permisos y herramientas](./agente/permisos-y-herramientas.md) y la [receta de reproducción con Codex](./agente/reproducibilidad.md) documentan la separación de componentes, las instrucciones y las verificaciones pendientes. Las pruebas negativas con PAT y su revocación siguen pendientes; no se atribuye un HTTP 403 no observado.

Esta tabla es la referencia canónica de entradas para el análisis LLM y el anexo. Permite reproducir las lecturas y verificar las citas con los objetos disponibles. La reproducción exacta de la salida del LLM requeriría, además, el contexto e instrucciones efectivos, la secuencia de salidas de herramientas, los parámetros de generación y la revisión exacta del modelo servido. Aun con ese registro, la identidad de la salida no queda garantizada.

## 8. Matriz de cumplimiento y pendientes del Issue #46

| Criterio | Estado histórico | Aporte de esta revisión y evidencia pendiente |
|---|---|---|
| Inventariar las entradas locales y producir diagnóstico 1.3.0 | Demostrado. El JSON contiene inventario, resumen y diagnostico_arquitectonico. | Los diez SHA identifican las entradas del análisis LLM. La identificación exacta de las entradas históricas sigue pendiente. |
| Contrastar la trazabilidad documental | Demostrado. Trece comprobaciones, incluida la discrepancia de HU-13. | El cotejo local confirma la discrepancia y la regla de anulación de DEC-06. Falta acreditar la revisión informada por QA en el PR #48. |
| Comparar alternativas y evaluar candidatos | Parcial. Tres alternativas comparadas y cuatro dominios no_evaluable. | La revisión caracteriza dependencias, invariantes y un candidato condicionado. La selección del piloto y su viabilidad operativa requieren mediciones comparables. |
| Operar en solo lectura y acreditar aislamiento efectivo | Parcial en el JSON. Declara solo_lectura. Los detalles de montajes, red y privilegios requieren el registro operativo separado. | Se conserva la inspección documentada del contenedor histórico y, por separado, el método de lectura local de esta sesión. Una repetición requiere evidencia propia de sus controles efectivos. [Permisos y herramientas](./agente/permisos-y-herramientas.md). |
| Adquisición paginada y verificación de una instantánea GitHub | Pendiente. El informe 1.3.0 no acredita adquisición remota ni procedencia GitHub en un informe 1.4.0. | La lectura local por SHA completa la identificación de los objetos analizados en esta revisión. La procedencia remota sigue pendiente. |
| Vincular issues de GitHub y cotejar árboles con commits autenticados | Pendiente. Las referencias documentales del JSON carecen de ese cotejo. | Falta evidencia de adquisición autenticada, vínculo con issues y correspondencia de los árboles con la fuente remota. |
| Verificar permisos efectivos y denegaciones de escritura con PAT | Pendiente. El JSON no contiene resultados con credenciales restringidas. | Se conserva pendiente el control positivo de lectura y la observación de las denegaciones de git push y gh issue comment en un entorno desechable autorizado. El 403 es el resultado esperado en el protocolo. Esta revisión no observó ninguna denegación. [Protocolo de pruebas negativas](./agente/prueba-negativa.md). |

### Validaciones operativas pendientes

- Datos y seguridad. Obtener evidencia redactada de las migraciones aplicadas, la identidad efectiva de sesión, membresías, propietario de funciones, contexto de auth.uid() y operaciones admitidas y rechazadas mediante la conexión real. spa-backend-api@2a0a2d41431f:app/compartido/db.py:19–44 · spa-database@971d27ca9059:supabase/migrations/20261107210000_movimientos.sql:61–65,224–251.
- Infraestructura. Vincular Vercel y Supabase a configuraciones y revisiones efectivas. Acreditar los artefactos ECR y, si se utiliza EKS, documentar el clúster, identidades, recursos aplicados, probes, exposición, mTLS, canario y reversión. DV5 y DV7 requieren evidencia de ejecución propia. spa-docs@aad24346c1cc:docs/02-arquitectura/arquitectura-servicios.md:324–345 · spa-infra-aws@e92e218c0518:terraform/envs/staging/main.tf:19–34 · spa-infra-aws@e92e218c0518:k8s/base/backend/deployment-v1.yaml:26–38.
- Rendimiento y evolución. Repetir Q5 con commits desplegados identificables, ampliar a carga mixta y sostenida, y comprobar integridad de inventario. El piloto de exportación depende de identificar un costo atribuible y de comparar su resultado con el monolito optimizado. spa-qa@9be19075107a5:suites/carga/q5_inventario_salidas.js:16–30,53–92 · spa-backend-api@2a0a2d41431f:tests/integration/movimientos/test_concurrencia_ultimo_stock.py:94–114 · spa-backend-api@2a0a2d41431f:app/modulos/analitica/infrastructure/exportador_duckdb.py:116–141.

La evidencia integrada sustenta la decisión arquitectónica del PR #47: mantener el monolito modular, preservar la autoridad transaccional de inventario y usar mediciones comparables antes de decidir una extracción. Los resultados históricos del motor, las conclusiones sobre código versionado y las comprobaciones operativas pendientes conservan su alcance propio
