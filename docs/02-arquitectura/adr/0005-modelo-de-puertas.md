# ADR-0005 · Puertas de integración (develop) y liberación certificada por QA (main)

- **Estado:** Aceptado
- **Fecha:** 2026-09-29
- **Reemplaza parcialmente:** ADR-0002 (reglas de aprobación)

## Contexto

Los equipos de backend, frontend y datos escriben el código de sus componentes y varios tienen un solo integrante; GitHub no permite aprobar el propio PR. Se requiere separar a quien construye de quien certifica, sin bloquear la integración diaria.

## Decisión

Rulesets por capas, que GitHub suma aplicando la regla más estricta:

1. `calidad-ramas-principales` (main y develop, todos los repos): PR obligatorio, checks en verde, historial lineal, sin force-push ni borrado. Bypass solo para plataforma-admins, vía PR.
2. `aprobacion-liberacion` (main de database, backend y frontend): una aprobación y re-aprobación tras cambios.
3. `solo-qa-promueve-a-main` (main de database, backend y frontend): solo equipo-qa puede actualizar main.

El bypass de QA vive aislado en la capa 3, de modo que QA sigue sujeto a checks y aprobación.

## Consecuencias

- Los autores integran en develop con su propio PR y el CI como red de seguridad.
- El paso a producción exige la certificación de QA (segregación de funciones).
- En los repos de plataforma el autor es DevOps; integran con PR y checks, sin aprobación humana obligatoria.
