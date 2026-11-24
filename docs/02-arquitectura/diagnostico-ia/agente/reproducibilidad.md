# Reproducción de la auditoría local

## Ejecución de referencia

| Dato | Valor |
|---|---|
| Motor y esquema de informe | `0.6.1.1` y `1.3.0` |
| Entorno | Docker Desktop `29.7.2`, motor Linux `amd64`, Python `3.12` |
| Imagen del auditor | `agente-auditor-ia:0.6.1.1` |
| ID de imagen inspeccionado | `sha256:c519aab85afd19d3ff0384cfca81c81a1e46c5ee92fad032c518a5e6f78a66ca` |
| Estado del contenedor al terminar | `exited`, código `0` |
| Evidencia conservada | `reports/diagnostico_arquitectura_20261123_232443_710295.json` |
| Cobertura | 10 de 10 repositorios, 301 archivos, 1.531 evidencias encontradas y clasificadas, 752 arquitectónicas, 13 hallazgos comprobados y una alerta informativa |

El informe registra `estado_informe=completo` y `estado_evidencia=parcial`. El primer estado corresponde a la cobertura de la ejecución. El segundo limita el alcance de las conclusiones arquitectónicas. El JSON no registra el commit del motor ni acredita revisiones Git reproducibles de los insumos. Esta receta reproduce el procedimiento y la configuración. Un resultado idéntico requiere preservar las mismas entradas y la misma imagen.

## Rutas y preparación

- Proyecto externo: copia local de `agente-auditor-ia`. Ejecutar los comandos desde la raíz de ese proyecto, donde están `docker/`, `repos/` y `reports/`.
- Entradas: `repos\<nombre>` en Windows y `/opt/auditor/repos/<nombre>` dentro del contenedor.
- Trazabilidad: `repos\spa-docs\docs\01-requisitos\TRAZABILIDAD.md`, visible para el motor bajo `/opt/auditor/repos/spa-docs/docs/01-requisitos/TRAZABILIDAD.md`.
- Salida: `reports\` en Windows y `/opt/auditor/reports` dentro del contenedor.
- Precondición: confirmar que existen los diez directorios, el documento y el directorio de salida. Mantener estables las entradas y comprobar que la salida está fuera de los árboles de repositorios.

## Comandos para una repetición autorizada

Ejecutar los bloques en orden y solo con autorización. Si la imagen del auditor ya está disponible y coincide con la configuración e identidad esperadas, usarla. Si se requiere reconstruir, comprobar la base fija en el motor local antes de redactar cualquier solicitud de descarga. En el bloque de construcción, `docker build --network=none` restringe la conectividad de los pasos de construcción. `--pull=false` evita solicitar explícitamente una versión más reciente de la imagen base. Ninguna de estas opciones ni la inspección previa de la base local demuestran por sí solas que BuildKit haya omitido todas las consultas externas. Para repetir el procedimiento sin conectividad, usar la imagen local previamente verificada o garantizar el aislamiento de red del entorno de construcción antes de reconstruir. Detenerse si falta la base o si el constructor intenta obtener recursos externos. Registrar una construcción como completamente sin conexión solo cuando exista evidencia suficiente.

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

Antes de arrancar, comprobar `created`, los 11 montajes y sus orígenes, los diez `RW=false`, el único `RW=true` en informes y los valores de aislamiento y recursos. Confirmar que `cap_add` sea `null` o una lista vacía (`[]`), lo que indica que no se añadieron capacidades. Revisar el entorno confinado en forma privada, sin volcar valores sensibles. Si difiere algún control obligatorio, detenerse. Con autorización separada de arranque:

```powershell
docker --host $motorIssue46 start --attach $contenedorIssue46
docker --host $motorIssue46 inspect --type container $contenedorIssue46 --format '{{.State.Status}} {{.State.ExitCode}}'
```

Conservar la consola y el código de salida. Con código `0`, identificar el nuevo `reports/diagnostico_arquitectura_*.json`, comprobar su lectura y cotejar esquema, cobertura y secciones arquitectónicas. La marca temporal en el nombre cambia en cada ejecución. No modificar permisos automáticamente si falla la escritura.

Fuentes del proyecto externo `agente-auditor-ia` (rutas relativas a su raíz): `config/MONTAJE-REPOSITORIOS.md`, `config/POLITICA-SEGURIDAD.md`, `docker/Dockerfile`, `agent/__main__.py`, `agent/version.py` y el JSON identificado arriba.

Refs #46
