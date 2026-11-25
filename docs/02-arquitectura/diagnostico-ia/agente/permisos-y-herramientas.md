# Permisos y herramientas del agente de IA — Issue #46

## 1. Componentes y alcance de seguridad

La solución utiliza dos componentes diferenciados:

- **Codex CLI:** agente basado en un modelo de lenguaje que analiza el código, contrasta evidencias y formula recomendaciones arquitectónicas.
- **Auditor Python:** herramienta complementaria de análisis estático, inventario y generación de resultados estructurados.

Los controles de seguridad de cada componente deben acreditarse por separado.

La configuración de Docker verificada para el auditor Python no demuestra automáticamente las restricciones de seguridad de Codex CLI.

## 2. Controles verificados en la ejecución local del auditor Python

La comprobación previa del contenedor `agente-auditor-issue46` registró estos valores mediante `docker inspect`. Se refieren al contenedor inspeccionado para esta ejecución.

| Control | Valor verificado |
|---|---|
| Imagen | `agente-auditor-ia:0.6.1.1`, `linux/amd64` |
| Entrada | `python -m agent` |
| Repositorios | Diez bind mounts individuales, todos `RW=false` |
| Salida | `/opt/auditor/reports`, único bind mount `RW=true`, con origen separado de los árboles auditados |
| Sistema raíz y red | `ReadonlyRootfs=true` y `NetworkMode=none` |
| Usuario y privilegios | `10001:10001`, modo privilegiado deshabilitado, `CapDrop=ALL`, sin capacidades añadidas y `no-new-privileges:true` |
| Recursos | 100 procesos, 256 MiB de memoria y 1 CPU |
| Entorno | Solo variables esperadas de la imagen, sin credenciales ni variables sensibles añadidas |

El arranque terminó con código `0` y se escribió el JSON en el montaje de informes.

El auditor tuvo acceso de lectura a los repositorios. La escritura de salida quedó circunscrita a `reports/`.

En una repetición, comprobar de nuevo la configuración y los permisos efectivos del directorio de salida.

## 3. Herramientas y límites del auditor Python

Docker Desktop prepara y aísla el contenedor. Python ejecuta el motor.

El CLI comprueba que la salida quede fuera de los árboles inspeccionados y crea el JSON mediante apertura exclusiva.

El inventario evita seguir los enlaces simbólicos que detecta y el buscador aplica redacción de patrones de secretos.

Estos controles se complementan con los montajes de solo lectura y la ausencia de red.

El contrato `1.0.0`, el lector local de instantáneas y el cotejo de árboles existen en el código.

El lector exige un digest del manifiesto obtenido por un canal confiable independiente.

El CLI que produjo el informe `1.3.0` no integra un extractor GitHub remoto ni necesita autenticarse ante GitHub.

## 4. Controles observados en la ejecución del agente Codex

La evaluación arquitectónica se realizó utilizando Codex CLI como agente basado en un modelo de lenguaje.

Cliente identificado en la sesión:

`OpenAI Codex v0.171.0-alpha.13`

Directorio de trabajo:

`C:\Users\mikec\agente-auditor-ia`

### 4.1. Configuración declarada

Durante la revisión, el entorno declaró:

- Acceso al sistema de archivos restringido a lectura.
- Política de aprobación `never`.
- Prohibición de operaciones de escritura.
- Prohibición de modificar repositorios, issues, pull requests o servicios.
- Prohibición de utilizar credenciales y secretos.

Estas restricciones fueron incluidas en las instrucciones del agente y se corresponden con el perfil declarado para la sesión.

**La declaración de una restricción no equivale a una prueba independiente de su imposición técnica.**

### 4.2. Herramientas utilizadas

Las operaciones observadas durante la evaluación fueron:

| Herramienta u operación | Finalidad | Evidencia |
|---|---|---|
| Enumeración de archivos | Identificar las fuentes disponibles | Observada |
| Lectura de archivos | Examinar código y documentación | Observada |
| Búsqueda de texto | Localizar implementaciones y referencias | Observada |
| `git ls-tree` | Consultar árboles de revisiones Git | Observada |
| `git show` | Consultar contenido de objetos Git | Observada |
| Interpretación mediante LLM | Contrastar evidencias y formular conclusiones | Observada en los resultados de la sesión |

La evaluación incluyó el cotejo de evidencia arquitectónica, historias de usuario, ADR, resultados de QA y dependencias entre módulos.

Como ejemplo, Codex identificó `q5-summary.json` en la raíz de `spa-qa`, contrastó sus métricas con el informe Q5 y corrigió una afirmación previa sobre la ausencia de dicho artefacto.

### 4.3. Operaciones no observadas

En el registro revisado no se observaron:

- Escrituras en los repositorios.
- Cambios de rama.
- Commits o publicaciones.
- Modificaciones de issues o pull requests.
- Solicitudes de elevación de permisos.
- Acceso remoto a servicios.
- Uso de credenciales.
- Ejecución de migraciones o despliegues.

Esta constatación describe únicamente las operaciones observadas durante la sesión. No constituye una garantía de imposibilidad de escritura.

### 4.4. Limitaciones de la evidencia

La sesión no acreditó mediante una comprobación independiente:

1. El rechazo efectivo de operaciones de escritura por el entorno de Codex.
2. El aislamiento de red a nivel de infraestructura.
3. Los permisos efectivos de una credencial GitHub.
4. La denegación HTTP 403 de operaciones remotas de escritura.
5. La revocación o caducidad del PAT.

Estas comprobaciones no deben confundirse con las pruebas de aislamiento realizadas anteriormente sobre el contenedor Docker del auditor Python.

**Estado del componente Codex:** ejecución de análisis LLM observada; configuración de solo lectura declarada y operaciones de lectura registradas; controles técnicos adicionales pendientes de acreditación.

## 5. Permisos GitHub propuestos para un extractor separado

| Permiso propuesto | Nivel | Uso |
|---|---|---|
| Metadata | Lectura | Resolver identidad de organización y repositorios. |
| Contents | Lectura | Resolver commits y leer árboles y documentos fijados a revisión. |
| Issues | Lectura | Leer issues mediante paginación. |
| Pull requests | Lectura | Leer PR mediante paginación. |

Para un futuro PAT fine-grained, el alcance propuesto se limita a repositorios seleccionados y una vigencia máxima de siete días.

Quedan fuera del alcance la escritura, administración, Actions, Workflows y Secrets.

La credencial corresponde al extractor separado. El auditor Python local opera sin credenciales.

La revisión realizada por Codex utilizó copias locales y objetos Git; no acredita por sí misma la configuración ni el funcionamiento de un extractor GitHub autenticado.

**Estado GitHub: propuesto y pendiente de verificación.**

Los permisos efectivos, la vigencia, las lecturas positivas y las denegaciones de escritura en GitHub carecen de evidencia de ejecución.

## 6. Pruebas negativas y cierre de credenciales

Las pruebas negativas se documentan en [prueba-negativa.md](./prueba-negativa.md).

Su objetivo es acreditar, mediante una ejecución separada y autorizada, que una identidad GitHub restringida puede realizar las lecturas permitidas y no puede ejecutar operaciones de escritura.

El protocolo debe registrar los resultados realmente observados, sin simular códigos HTTP ni atribuir a la configuración restricciones que no hayan sido verificadas.

La revocación o caducidad del PAT también requiere evidencia documental.

**Estado: pendiente de ejecución.**

## 7. Fuentes y reproducibilidad

Fuentes del proyecto externo `agente-auditor-ia`, con rutas relativas a su raíz:

- `config/MONTAJE-REPOSITORIOS.md`
- `config/POLITICA-SEGURIDAD.md`
- `docker/Dockerfile`
- `.dockerignore`
- `agent/__main__.py`
- `agent/inspector/inventario.py`
- `agent/inspector/buscador.py`
- `agent/inspector/instantanea_github.py`

Documentación relacionada:

- [Prompt y ejecución del agente](./prompt.md).
- [Reproducibilidad](./reproducibilidad.md).
- [Pruebas negativas](./prueba-negativa.md).
- [Diagnóstico arquitectónico](../diagnostico-arquitectura.md).
- [Análisis técnico LLM](../analisis-tecnico-llm.md).

Refs #46