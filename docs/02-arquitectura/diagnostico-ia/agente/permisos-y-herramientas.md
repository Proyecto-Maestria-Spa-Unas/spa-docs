# Permisos y herramientas del auditor

## Controles verificados en la ejecución local

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

El arranque terminó con código `0` y se escribió el JSON en el montaje de informes. El auditor tuvo acceso de lectura a los repositorios. La escritura de salida quedó circunscrita a `reports/`. En una repetición, comprobar de nuevo la configuración y los permisos efectivos del directorio de salida.

## Herramientas y límites del código local

Docker Desktop prepara y aísla el contenedor. Python ejecuta el motor. El CLI comprueba que la salida quede fuera de los árboles inspeccionados y crea el JSON mediante apertura exclusiva. El inventario evita seguir los enlaces simbólicos que detecta y el buscador aplica redacción de patrones de secretos. Estos controles se complementan con los montajes de solo lectura y la ausencia de red.

El contrato `1.0.0`, el lector local de instantáneas y el cotejo de árboles existen en el código. El lector exige un digest del manifiesto obtenido por un canal confiable independiente. El CLI que produjo el informe `1.3.0` no integra un extractor GitHub remoto ni necesita autenticarse ante GitHub.

## Permisos GitHub propuestos para un extractor separado

| Permiso propuesto | Nivel | Uso |
|---|---|---|
| Metadata | Lectura | Resolver identidad de organización y repositorios. |
| Contents | Lectura | Resolver commits y leer árboles y documentos fijados a revisión. |
| Issues | Lectura | Leer issues mediante paginación. |
| Pull requests | Lectura | Leer PR mediante paginación. |

Para un futuro PAT fine-grained, el alcance propuesto se limita a repositorios seleccionados y una vigencia máxima de siete días. Quedan fuera del alcance la escritura, administración, Actions, Workflows y Secrets. La credencial corresponde al extractor separado. El auditor local opera sin credenciales.

**Estado GitHub: propuesto y pendiente de verificación.** La configuración del contenedor quedó verificada para la ejecución local. Los permisos efectivos, la vigencia, las lecturas positivas y las denegaciones de escritura en GitHub carecen de evidencia de ejecución.

Fuentes del proyecto externo `agente-auditor-ia` (rutas relativas a su raíz): `config/MONTAJE-REPOSITORIOS.md`, `config/POLITICA-SEGURIDAD.md`, `docker/Dockerfile`, `.dockerignore`, `agent/__main__.py`, `agent/inspector/inventario.py`, `agent/inspector/buscador.py` y `agent/inspector/instantanea_github.py`.

Refs #46
