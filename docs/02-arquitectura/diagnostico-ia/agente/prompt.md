# Prompt y ejecución del agente de IA — Issue #46

## 1. Componentes de la solución

La solución de auditoría arquitectónica SPA-UNAS utiliza dos componentes complementarios:

**Auditor Python estático:** inventaría repositorios, identifica patrones, compara matrices de trazabilidad y genera un diagnóstico estructurado en JSON. No utiliza un modelo de lenguaje en su flujo de ejecución verificado.

**Agente Codex CLI:** utiliza un modelo de lenguaje para interpretar instrucciones, seleccionar herramientas de lectura, contrastar código y documentación, evaluar alternativas arquitectónicas y producir recomendaciones justificadas.

Codex CLI es el componente de inteligencia artificial utilizado para la evaluación arquitectónica del Issue #46. El auditor Python se conserva como herramienta complementaria de recopilación de evidencias.

Los resultados se presentan en:

- [Diagnóstico arquitectónico](../diagnostico-arquitectura.md).
- [Análisis técnico mediante LLM](../analisis-tecnico-llm.md).

## 2. Registro histórico del auditor Python

**Prompt histórico de lenguaje natural: sin evidencia disponible.**

En el punto de entrada, el consolidador y la configuración Docker revisados no hay un prompt de modelo ni un parámetro para suministrarlo. El informe JSON tampoco registra un prompt.

La orden reproducible de la ejecución fue `python -m agent`, que es un comando de entrada, no un prompt de lenguaje natural.

### Qué ejecutó el auditor

1. Validó las rutas locales de repositorios, trazabilidad y salida.
2. Inventarió los repositorios, buscó patrones y clasificó las coincidencias.
3. Extrajo y comparó las matrices documentales y los endpoints.
4. Construyó el diagnóstico arquitectónico, separando los hallazgos documentales de las inferencias y las alternativas con evidencia insuficiente.
5. Validó el informe `1.3.0` antes de escribir el JSON en `reports/`.

La ejecución registrada usó el motor `0.6.1.1` y el modo `solo_lectura`.

Incluyó 10 repositorios, 301 archivos y 1.531 evidencias clasificadas.

Los 13 hallazgos comprobados corresponden a la comparación de matrices. La evidencia arquitectónica quedó `parcial`, y los cuatro dominios examinados quedaron `no_evaluable`.

El informe no acredita la implementación de una historia de usuario por su sola presencia documental ni justifica por sí mismo un piloto de extracción.

### Fuentes locales

Las rutas son relativas a la raíz del proyecto externo `agente-auditor-ia`.

- `docker/Dockerfile`: define el comando de entrada.
- `agent/__main__.py`: valida rutas y coordina la ejecución.
- `agent/inspector/consolidador.py`: consolida y valida la salida.
- `reports/diagnostico_arquitectura_20261123_232443_710295.json`: registra el resultado observado.

## 3. Ejecución de Codex como agente LLM

La evaluación de cierre se realizó mediante una sesión identificada como:

**OpenAI Codex CLI v0.171.0-alpha.13**

Directorio de trabajo:

`C:\Users\mikec\agente-auditor-ia`

Codex recibió instrucciones en lenguaje natural, consultó archivos y objetos Git, contrastó evidencias y formuló conclusiones arquitectónicas.

Durante la revisión independiente identificó `q5-summary.json` en la raíz de `spa-qa`, contrastó sus métricas con el informe Q5 y corrigió una afirmación anterior sobre la ausencia de dicho artefacto.

Este comportamiento constituye evidencia de análisis mediante LLM y no únicamente de clasificación estática por patrones.

### Controles de seguridad observados

La sesión declaró:

- Acceso al sistema de archivos restringido a lectura.
- Política de aprobación `never`.
- Uso de herramientas para enumeración, búsqueda y lectura de archivos.
- Consultas de objetos Git mediante herramientas de lectura.

No se observaron operaciones de escritura ni utilización de credenciales durante la evaluación.

**Limitaciones:** esta observación no demuestra por sí sola el rechazo efectivo de operaciones de escritura, el aislamiento de red a nivel de infraestructura ni los permisos efectivos de una credencial GitHub.

Las pruebas negativas y la revocación del PAT permanecen pendientes y se documentan en [prueba-negativa.md](./prueba-negativa.md).

## 4. Prompt maestro de evaluación arquitectónica

El siguiente prompt se utilizó para la evaluación de cierre del agente Codex. Los análisis anteriores también dependieron del contexto y las instrucciones de sus respectivas sesiones.

### Rol

Actúa como un agente de inteligencia artificial especializado en arquitectura de software, sistemas distribuidos, arquitectura hexagonal, escalabilidad, seguridad, DevSecOps, ingeniería de requisitos y evaluación de microservicios.

Tu motor de razonamiento es el modelo de lenguaje utilizado por Codex CLI.

**Objetivo:** evaluar el cumplimiento técnico del Issue #46 de SPA-UNAS, reutilizando el auditor Python y las evidencias existentes, sin reconstruir componentes innecesariamente.

### Restricción obligatoria: solo lectura

Opera exclusivamente mediante herramientas de lectura.

Está prohibido crear, modificar o eliminar archivos; ejecutar operaciones de escritura en Git; modificar issues o pull requests; ejecutar migraciones o despliegues; acceder a secretos o credenciales; modificar bases de datos o infraestructura; y ejecutar comandos con efectos secundarios.

Si una operación requiere escritura, detente y explica la autorización necesaria. No la ejecutes.

### Contexto y fuentes

Analiza el proyecto SPA-UNAS, basado en FastAPI, arquitectura hexagonal, PostgreSQL/Supabase, frontend web, Docker y componentes de infraestructura.

Los dominios principales son Identidad, Catálogo, Movimientos de inventario y Analítica.

Directorio de trabajo:

`C:\Users\mikec\agente-auditor-ia`

Repositorios:

`C:\Users\mikec\agente-auditor-ia\repos`

Examina los diez repositorios disponibles, con especial atención a:

- `spa-docs/docs/01-requisitos/`
- `spa-docs/docs/01-requisitos/TRAZABILIDAD.md`
- `spa-docs/docs/02-arquitectura/`
- `spa-backend-api/app/main.py`
- ADR-0009 y ADR-0010.
- Acceso a datos y transacciones.
- Configuración de Supabase.
- Infraestructura AWS y Vercel.
- Pruebas de rendimiento y calidad.

No supongas que un componente está implementado porque aparece mencionado en un documento.

### Aprovechamiento del trabajo existente

Utiliza el auditor Python como fuente complementaria de inventario, clasificación y trazabilidad.

Consulta el diagnóstico arquitectónico, el anexo técnico LLM, la documentación de permisos, el procedimiento de reproducibilidad y el protocolo de pruebas negativas.

No reconstruyas el auditor ni repitas análisis que ya dispongan de evidencia suficiente.

### Evaluación arquitectónica

**Arquitectura actual:** identifica módulos, responsabilidades, dependencias, acoplamiento, límites transaccionales, propiedad de datos y riesgos.

**Historias de usuario:** contrasta requisitos, código y matriz de trazabilidad. Clasifica cada resultado como verificado, parcialmente verificado, no verificado o no evaluable.

**Escalabilidad:** evalúa base de datos compartida, pooler de Supabase, API serverless en Vercel, consultas de Analítica, broker de trabajos, ADR-0010 y propuesta EKS de ADR-0009.

**Seguridad y DevSecOps:** examina JWT, autorización, RLS, secretos, CI/CD, contenedores, dependencias y aislamiento. Distingue configuración versionada de controles efectivamente operativos.

**Microservicios:** determina si conviene mantener el monolito modular, qué componentes podrían extraerse, en qué orden, bajo qué criterios y con qué riesgos.

Considera especialmente la consistencia del inventario, los triggers, los bloqueos `FOR UPDATE`, los costos operativos y las métricas necesarias para justificar una extracción.

No asumas que microservicios es la solución correcta.

### Reglas de evidencia

Cada hallazgo debe incluir:

- Repositorio.
- Archivo y línea, cuando estén disponibles.
- Evidencia observada.
- Interpretación técnica.
- Nivel de confianza.
- Limitaciones.

Distingue expresamente entre hechos verificados, inferencias y afirmaciones no verificadas.

No inventes métricas, pruebas, configuraciones ni referencias.

### Verificación del agente LLM

Identifica el modelo y la versión disponibles, el prompt utilizado, las herramientas habilitadas, las restricciones de solo lectura, las revisiones Git, las evidencias consultadas y el procedimiento reproducible.

No afirmes que existe una integración Python con LLM si no está implementada.

### Criterios de aceptación del Issue #46

Evalúa individualmente:

1. Credencial GitHub restringida a lectura y vigencia máxima de siete días.
2. Herramientas del agente sin capacidad de modificación.
3. Pruebas negativas de escritura con HTTP 403.
4. Cobertura del diagnóstico arquitectónico.
5. Propuesta fundamentada sobre microservicios.
6. Procedimiento reproducible.
7. PR sin secretos y con referencia al Issue #46.
8. Revocación o vencimiento documentado del token.

Clasifica cada criterio como cumplido, parcialmente cumplido, pendiente o no verificable.

No ejecutes ni simules las pruebas negativas de GitHub durante esta evaluación de lectura.

### Entregables

Presenta:

**A.** Diagnóstico del estado real del agente LLM.

**B.** Matriz de cumplimiento del Issue #46.

**C.** Brechas técnicas pendientes.

**D.** Plan mínimo de corrección.

**E.** Conclusión sobre la viabilidad de Codex CLI, la necesidad o no de otro agente Python, los componentes reutilizables y el siguiente paso indispensable.

No modifiques archivos, no publiques cambios en GitHub y no declares cumplidos requisitos pendientes.

## 5. Resultados del agente Codex

La evaluación concluyó que Codex CLI puede utilizarse como agente LLM para el diagnóstico arquitectónico solicitado.

La recomendación técnica fue mantener el monolito modular, conservando una autoridad transaccional compartida para Catálogo y Movimientos.

La posible extracción de Analítica o exportaciones debe condicionarse a mediciones de carga mixta, identificación de cuellos de botella y evaluación de un piloto reversible.

La tabla canónica de revisiones Git se encuentra en el [diagnóstico arquitectónico](../diagnostico-arquitectura.md). La argumentación detallada se encuentra en el [análisis técnico LLM](../analisis-tecnico-llm.md).

## 6. Reproducibilidad y pendientes

La reproducción de la evaluación requiere conservar el prompt, las revisiones Git, el contexto relevante de sesión, las herramientas disponibles y las evidencias consultadas.

El procedimiento se desarrolla en [reproducibilidad.md](./reproducibilidad.md).

La repetición de las mismas lecturas permite comprobar las referencias, pero no garantiza que un modelo de lenguaje genere una respuesta idéntica.

**Estado:** el uso de Codex como agente LLM queda documentado mediante la sesión y sus resultados. Las pruebas negativas del PAT, su revocación y la acreditación integral de los controles de seguridad siguen pendientes.

Refs #46