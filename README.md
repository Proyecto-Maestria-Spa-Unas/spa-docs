# Documentación Spa de Uñas - Requisitos, Historias y Arquitectura

Repositorio de documentación del sistema de gestión y control interno del inventario de un spa de uñas. Reúne la especificación de requisitos, las historias de usuario, la matriz de trazabilidad, las decisiones de arquitectura (ADR) y los procesos de trabajo del equipo.

## 🚀 Contenido Principal

* Especificación de requisitos funcionales (RF-01 a RF-20)
* Requisitos no funcionales (RNF-01 a RNF-10)
* Reglas de negocio (RN-01 a RN-14)
* Historias de usuario (HU-01 a HU-13) con criterios de aceptación
* Decisiones de arquitectura (ADR)
* Flujo de trabajo Git y Definición de Terminado

## ⚙️ Cómo trabajar con este repositorio

### 1️⃣ Clonar el repositorio

```
git clone https://github.com/Proyecto-Maestria-Spa-Unas/spa-docs.git
cd spa-docs
```

### 2️⃣ Editar

La documentación se escribe en Markdown. Los diagramas se escriben con Mermaid dentro de los mismos archivos y GitHub los muestra directamente.

### 3️⃣ Proponer cambios

```
git checkout -b docs/descripcion-del-cambio
git commit -s -m "docs(requisitos): actualizar criterios de HU-07"
git push -u origin docs/descripcion-del-cambio
```

Luego abra un Pull Request hacia `main`. Lo aprueba el equipo de analistas.

## 📋 Historias de usuario y decisiones pendientes

Las historias de usuario no se editan como archivos: viven como **issues** de este repositorio con la etiqueta `historia-usuario`, organizadas en los milestones M0 a M5 y en el tablero del proyecto de la organización.

Las decisiones pendientes de la especificación (rol Gerente, permisos de la Manicurista, alertas obligatorias, filtro por fechas, eliminación de movimientos, métricas de rendimiento, políticas de contraseñas, respaldo y catálogo definitivo) están como issues con la etiqueta `decision-pendiente` (DEC-01 a DEC-10). Al resolver una, se registra su ADR y se cierra el issue.

## 📁 Organización del proyecto

| Carpeta | Uso |
|---|---|
| `docs/01-requisitos` | Especificación oficial y matriz de trazabilidad HU ↔ RF/RNF/RN ↔ repositorios. |
| `docs/02-arquitectura/adr` | Registros de decisiones de arquitectura (usar `0000-plantilla.md`). |
| `docs/03-procesos` | Flujo de trabajo Git, releases y Definición de Terminado. |

## 🏛️ Decisiones registradas

| ADR | Estado | Tema |
|---|---|---|
| 0001 | Aceptado | Organización GitHub con un repositorio por componente |
| 0002 | Aceptado | Flujo de ramas y puerta de QA antes del merge |
| 0003 | Propuesto | Anulación controlada de movimientos en lugar de eliminación |
| 0004 | Aceptado | PyJWT en lugar de python-jose para JWT |
| 0005 | Aceptado | Puertas de integración (develop) y liberación certificada por QA (main) |
| 0006 | Aceptado | Squash para integrar, merge commit para liberar |

## 🌿 Cómo contribuir

### 1️⃣ Clonar el repositorio (solo la primera vez)

Use **Git Bash** en Windows o la terminal en Linux/Mac:

```
git config --global core.autocrlf input
git clone https://github.com/Proyecto-Maestria-Spa-Unas/spa-docs.git
cd spa-docs
git switch main
```

### 2️⃣ Crear la rama de su tarea

Nunca se trabaja directamente sobre `main` ni `develop`: GitHub rechaza esos push. Cada tarea tiene su rama, creada desde `main` actualizada:

```
git switch main
git pull
git switch -c docs/F1-investigacion-marca
```

Formato obligatorio: **`tipo/ID-descripcion-corta`**. El `ID` es el de la tarea del sprint en mayúscula (D1, B2, F3, Q1…) y la descripción va en minúsculas, con guiones y sin espacios ni tildes.

| Tipo | Úselo para |
|---|---|
| `feature/` | Funcionalidad nueva |
| `fix/` | Corrección de un defecto |
| `docs/` | Documentación |
| `test/` | Pruebas |
| `refactor/` | Mejora interna sin cambio funcional |
| `chore/` · `ci/` | Mantenimiento y automatización |

### 3️⃣ Guardar y subir los cambios

```
git add .
git commit -s -m "docs(marca): investigación de marca y benchmark (F1)"
git push -u origin docs/F1-investigacion-marca
```

El mensaje sigue **Conventional Commits**: `tipo(alcance): descripción (ID)`. La opción `-s` firma el commit.

### 4️⃣ Abrir el Pull Request

```
gh pr create --base main --fill
```

O desde GitHub con el botón **Compare & pull request**. En la descripción escriba `Closes Proyecto-Maestria-Spa-Unas/spa-docs#<número de la tarea>`. El PR se fusiona cuando los checks obligatorios están en verde.

### 5️⃣ Mantener su rama al día

Si `main` avanzó mientras usted trabajaba:

```
git switch main
git pull
git switch -
git rebase main
git push --force-with-lease
```

`--force-with-lease` solo se usa sobre **su propia rama**, nunca sobre `main` ni `develop`.

### ❌ Qué no hacer

* No subir archivos `.env`, contraseñas ni llaves: el escaneo de seguridad bloqueará el PR.
* No mezclar varias tareas en una misma rama: una rama, una tarea, un PR.
