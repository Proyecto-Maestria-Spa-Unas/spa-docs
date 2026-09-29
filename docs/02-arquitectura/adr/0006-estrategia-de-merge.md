# ADR-0006 · Squash para integrar, merge commit para liberar

- **Estado:** Aceptado
- **Fecha:** 2026-09-29
- **Complementa:** ADR-0005

## Contexto

Con solo squash, cada liberación develop → main crearía en main un commit que no existe en develop. Las ramas divergirían y cada PR de liberación arrastraría historia repetida y conflictos falsos.

## Decisión

- `develop`: solo squash e historial lineal (ruleset `integracion-develop`). Un commit por historia de usuario.
- `main` en repos con develop: solo merge commit (ruleset `liberacion-main`). Cada liberación queda marcada y main contiene los mismos commits de develop.
- `main` en repos sin develop: solo squash e historial lineal.
- En database, backend y frontend, la liberación requiere la aprobación de equipo-qa (`required_reviewers`).

## Consecuencias

- develop y main no divergen; los PR de liberación muestran solo lo nuevo.
- La historia de main distingue liberaciones (merge commits) de cambios (commits de develop).
