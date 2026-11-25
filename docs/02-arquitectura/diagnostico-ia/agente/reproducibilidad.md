# Reproducibilidad del agente de IA — Issue #46

## 1. Alcance

La solución SPA-UNAS utiliza dos procedimientos complementarios:

1. **Auditor Python:** ejecución estática en Docker, con generación de un informe JSON.
2. **Agente Codex CLI:** análisis mediante LLM de las evidencias, el código, las historias de usuario y las alternativas arquitectónicas.

La reproducción de uno no sustituye la reproducción del otro.

Los resultados de ambos componentes se relacionan en el [diagnóstico arquitectónico](../diagnostico-arquitectura.md) y el [análisis técnico LLM](../analisis-tecnico-llm.md).

## 2. Ejecución de referencia del auditor Python

| Dato | Valor |
|---|---|
| Motor y esquema de informe | `0.6.1.1` y `1.3.0` |
| Entorno | Docker Desktop `29.7.2`, motor Linux `amd64`, Python `3.12` |
| Imagen del auditor | `agente-auditor-ia:0.6.1.1` |
| ID de imagen inspeccionado | `sha256:c519aab85afd19d3ff0384cfca81c81a1e46c5ee92fad032c518a5e6f78a66ca` |
| Estado del contenedor al terminar | `exited`, código `0` |
| Evidencia conservada | `reports/diagnostico_arquitectura_20261123_232443_710295.json` |
| Cobertura | 10 de 10 repositorios, 301 archivos, 1.531 evidencias encontradas y clasificadas, 752 arquitectónicas, 13 hallazgos comprobados y una alerta informativa |

El informe registra `estado_informe=completo` y `estado_evidencia=parcial`.

El primer estado corresponde a la cobertura de la ejecución. El segundo limita el alcance de las conclusiones arquitectónicas.

El JSON no registra el commit del motor ni acredita revisiones Git reproducibles de los insumos.

Esta receta reproduce el procedimiento y la configuración. Un resultado idéntico requiere preservar las mismas entradas y la misma imagen.

## 3. Rutas y preparación del auditor Python

- Proyecto externo: copia local de `agente-auditor-ia`. Ejecutar los comandos desde la raíz de ese proyecto, donde están `docker/`, `repos/` y `reports/`.
- Entradas: `repos\<nombre>` en Windows y `/opt/auditor/repos/<nombre>` dentro del contenedor.
- Trazabilidad: `repos\spa-docs\docs\01-requisitos\TRAZABILIDAD.md`, visible para el motor bajo `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md`.
- Salida: `reports\` en Windows y `/opt/auditor/reports` dentro del contenedor.
- Precondición: confirmar que existen los diez directorios, el documento y el directorio de salida. Mantener estables las entradas y comprobar que la salida está fuera de los árboles de repositorios.

## 4. Comandos para una repetición autorizada del auditor Python

Ejecutar los bloques en orden y solo con autorización.

Si la imagen del auditor ya está disponible y coincide con la configuración e identidad esperadas, usarla.

Si se requiere reconstruir, comprobar la base fija en el motor local antes de redactar cualquier solicitud de descarga.

En el bloque de construcción, `docker build --network=none` restringe la conectividad de los pasos de construcción. `--pull=false` evita solicitar explícitamente una versión más reciente de la imagen base.

Ninguna de estas opciones ni la inspección previa de la base local demuestran por sí solas que BuildKit haya omitido todas las consultas externas.

Para repetir el procedimiento sin conectividad, usar la imagen local previamente verificada o garantizar el aislamiento de red del entorno de construcción antes de reconstruir.

Detenerse si falta la base o si el constructor intenta obtener recursos externos.

Registrar una construcción como completamente sin conexión solo cuando exista evidencia suficiente.

```powershell
$raizIssue46 = (Get-Location).Path
$motorIssue46 = 'npipe:////./pipe/dockerDesktopLinuxEngine'
$imagenIssue46 = 'agente-auditor-ia:0.6.1.1'
$baseIssue46 = 'python:3.12-slim@sha256:5e2bdb1b0d542d5039cffb8181c65930093949194a65c365dfb1b0fe13a4fa5b'
```

Este bloque es opcional. Ejecutarlo solo si hace falta reconstruir la imagen.

```powershell
docker --host $motorIssue46 image inspect $baseIssue46
if ($LASTEXITCODE -ne 0) { throw 'Falta la base local fijada' }
docker --host $motorIssue46 build --platform linux/amd64 --network=none --pull=false --file (Join-Path $raizIssue46 'docker\Dockerfile') --tag $imagenIssue46 $raizIssue46
if ($LASTEXITCODE -ne 0) { throw 'Falló la construcción' }
```

Crear un contenedor nuevo solo para una repetición autorizada. El nombre de este ejemplo debe estar libre. `--pull never` impide que la creación obtenga otra imagen.

```powershell
docker --host $motorIssue46 image inspect $imagenIssue46 --format '{{.Id}} {{.Os}}/{{.Architecture}} {{json .Config.Cmd}} {{.Config.User}}'
if ($LASTEXITCODE -ne 0) { throw 'Falta la imagen local del auditor' }

$reposIssue46 = @('.github','spa-backend-api','spa-database','spa-devops-workflows','spa-docs','spa-frontend-web','spa-infra-aws','spa-org-bootstrap','spa-qa','spa-template-servicio')
$argumentosIssue46 = @('--platform','linux/amd64','--name','agente-auditor-issue46-reproduccion','--pull','never','--network','none','--read-only','--cap-drop','ALL','--security-opt','no-new-privileges:true','--pids-limit','100','--memory','256m','--cpus','1','--user','10001:10001')
foreach ($repoIssue46 in $reposIssue46) {
    $origenIssue46 = Join-Path $raizIssue46 "repos\$repoIssue46"
    $argumentosIssue46 += @('--mount',"type=bind,source=$origenIssue46,target=/opt/auditor/repos/$repoIssue46,readonly")
}
$salidaIssue46 = Join-Path $raizIssue46 'reports'
$argumentosIssue46 += @('--mount',"type=bind,source=$salidaIssue46,target=/opt/auditor/reports",$imagenIssue46)
$contenedorIssue46 = docker --host $motorIssue46 create @argumentosIssue46
if ($LASTEXITCODE -ne 0) { throw 'Falló la creación' }

docker --host $motorIssue46 inspect --type container $contenedorIssue46 --format '{{.State.Status}} {{json .Mounts}}'
docker --host $motorIssue46 inspect --type container $contenedorIssue46 --format 'red={{.HostConfig.NetworkMode}} rootfs_ro={{.HostConfig.ReadonlyRootfs}} usuario={{.Config.User}} privilegiado={{.HostConfig.Privileged}} cap_drop={{json .HostConfig.CapDrop}} cap_add={{json .HostConfig.CapAdd}} seguridad={{json .HostConfig.SecurityOpt}} pids={{.HostConfig.PidsLimit}} memoria={{.HostConfig.Memory}} cpu_nano={{.HostConfig.NanoCpus}}'
```

Antes de arrancar, comprobar `created`, los 11 montajes y sus orígenes, los diez `RW=false`, el único `RW=true` en informes y los valores de aislamiento y recursos.

Confirmar que `cap_add` sea `null` o una lista vacía (`[]`), lo que indica que no se añadieron capacidades.

Revisar el entorno confinado en forma privada, sin volcar valores sensibles.

Si difiere algún control obligatorio, detenerse.

Con autorización separada de arranque:

```powershell
docker --host $motorIssue46 start --attach $contenedorIssue46
docker --host $motorIssue46 inspect --type container $contenedorIssue46 --format '{{.State.Status}} {{.State.ExitCode}}'
```

Conservar la consola y el código de salida.

Con código `0`, identificar el nuevo `reports/diagnostico_arquitectura_*.json`, comprobar su lectura y cotejar esquema, cobertura y secciones arquitectónicas.

La marca temporal en el nombre cambia en cada ejecución.

No modificar permisos automáticamente si falla la escritura.

## 5. Reproducción de la revisión arquitectónica con Codex CLI

### 5.1. Identificación de la ejecución

| Dato | Registro |
|---|---|
| Cliente | OpenAI Codex CLI |
| Versión observada | `0.171.0-alpha.13` |
| Modelo informado para la entrega | `GPT-6.2-Sol` |
| Directorio de trabajo | `C:\Users\mikec\agente-auditor-ia` |
| Modo declarado | Sistema de archivos restringido a lectura |
| Política de aprobación declarada | `never` |
| Fuentes | Diez repositorios SPA-UNAS |
| Prompt | [Prompt maestro](./prompt.md) |
| Resultados | Diagnóstico arquitectónico y anexo técnico LLM |

Los identificadores del cliente y del modelo se conservan como metadatos de sesión.

La identidad exacta del binario y la revisión interna del modelo servido no se acreditaron de manera independiente.

### 5.2. Preparación de las entradas

Para repetir la evaluación:

1. Disponer de las copias locales de los diez repositorios SPA-UNAS.
2. Conservar los objetos Git correspondientes a las revisiones analizadas.
3. Consultar la tabla canónica de SHA del [diagnóstico arquitectónico](../diagnostico-arquitectura.md).
4. Mantener disponible el JSON histórico del auditor Python.
5. Conservar el prompt maestro y las instrucciones adicionales relevantes de la evaluación original.

Los SHA de los repositorios constituyen la referencia para comprobar las citas.

La rama activa puede contener cambios posteriores; por eso, las referencias deben resolverse contra los objetos Git fijados.

### 5.3. Configuración de seguridad

Abrir Codex CLI desde la raíz de `agente-auditor-ia`, utilizando un perfil que restrinja el acceso al sistema de archivos a operaciones de lectura y una política de aprobación `never`.

No habilitar herramientas de escritura, operaciones de publicación ni acceso a credenciales.

No utilizar secretos de Supabase, Vercel, AWS o Azure.

Las copias de repositorios deben tratarse como fuentes de consulta.

**La configuración efectiva de seguridad debe comprobarse en cada nueva ejecución.** La declaración del prompt no sustituye la imposición técnica de restricciones.

El entorno de Codex no debe confundirse con el contenedor Docker del auditor Python.

### 5.4. Consulta de revisiones Git

Para identificar los archivos de una revisión fijada:

```powershell
git -C "C:\Users\mikec\agente-auditor-ia\repos\spa-backend-api" ls-tree -r --name-only <SHA>
```

Para consultar el contenido de un archivo en una revisión:

```powershell
git -C "C:\Users\mikec\agente-auditor-ia\repos\spa-backend-api" show "<SHA>:app/main.py"
```

En ambos ejemplos, `<SHA>` debe reemplazarse por el identificador real registrado en la tabla canónica del diagnóstico.

Estas operaciones consultan objetos Git sin modificar los repositorios.

Aplicar el mismo procedimiento a las demás fuentes y revisiones documentadas.

### 5.5. Ejecución del análisis LLM

1. Iniciar una sesión de Codex CLI con los controles de lectura establecidos.
2. Proporcionar el prompt maestro documentado en `prompt.md`.
3. Suministrar las revisiones Git y el contexto necesario para interpretar las fuentes.
4. Permitir únicamente herramientas de lectura.
5. Solicitar el cotejo de hechos, inferencias y limitaciones.
6. Contrastar las conclusiones con el diagnóstico y el anexo técnico.
7. Conservar el registro de la sesión, las instrucciones y las referencias verificables.

Para reconstruir también la secuencia histórica de correcciones es necesario conservar las instrucciones previas pertinentes y las salidas efectivamente recibidas por el modelo.

### 5.6. Verificación de resultados

La persona que reproduzca la evaluación debe comprobar:

- Correspondencia entre repositorios, SHA y archivos citados.
- Existencia de las evidencias utilizadas.
- Coherencia de las referencias a líneas.
- Separación entre hechos verificados e inferencias.
- Cobertura de arquitectura, historias de usuario, escalabilidad, seguridad y microservicios.
- Identificación explícita de resultados no verificados.
- Ausencia de modificaciones a los repositorios durante la revisión.

La repetición permite verificar las fuentes y reconstruir el procedimiento de análisis.

**No garantiza una salida textual idéntica**, porque la generación mediante un modelo de lenguaje puede variar entre ejecuciones.

## 6. Límites y requisitos pendientes

La evidencia histórica del auditor Python no identifica los SHA exactos de todas sus entradas.

La evaluación con Codex utiliza revisiones Git fijadas, pero la reconstrucción completa de su contexto requiere conservar los turnos previos relevantes, las salidas consultadas y los parámetros disponibles del modelo.

La ejecución de Codex no sustituye las pruebas de permisos GitHub.

Permanecen pendientes:

- Acreditar permisos efectivos y vigencia del PAT.
- Ejecutar las pruebas negativas autorizadas y registrar los resultados observados.
- Documentar la revocación o caducidad del PAT.
- Verificar la configuración efectiva de seguridad en una nueva ejecución reproducible.
- Cotejar el diff final del PR #47 y sus evidencias de seguridad.

Estos requisitos se gestionan separadamente mediante [prueba-negativa.md](./prueba-negativa.md).

## 7. Fuentes

Fuentes del proyecto externo `agente-auditor-ia`, con rutas relativas a su raíz:

- `config/MONTAJE-REPOSITORIOS.md`
- `config/POLITICA-SEGURIDAD.md`
- `docker/Dockerfile`
- `agent/__main__.py`
- `agent/version.py`
- `reports/diagnostico_arquitectura_20261123_232443_710295.json`

Documentos relacionados:

- [Prompt y ejecución del agente](./prompt.md).
- [Permisos y herramientas](./permisos-y-herramientas.md).
- [Pruebas negativas](./prueba-negativa.md).
- [Diagnóstico arquitectónico](../diagnostico-arquitectura.md).
- [Análisis técnico LLM](../analisis-tecnico-llm.md).

Refs #46