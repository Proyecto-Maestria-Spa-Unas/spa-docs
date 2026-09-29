# ADR-0004 · PyJWT en lugar de python-jose para tokens JWT

- **Estado:** Aceptado
- **Fecha:** 2026-09-29
- **Relacionado:** HU-01 · RNF-01 · PYSEC-2026-1325 (CVE-2024-23342, GHSA-wj6h-64fc-37mp)

## Contexto

El stack original definía `python-jose[cryptography]` para JWT. Esa librería instala como dependencia transitiva `ecdsa`, afectada por un ataque de temporización (Minerva) sobre P-256 para el que el proyecto upstream declaró que no habrá corrección. La auditoría `pip-audit` del pipeline `calidad / pipeline` fallaba de forma permanente por esta causa.

## Decisión

Reemplazar `python-jose` por `PyJWT` en `spa-backend-api` y en `spa-template-servicio`, incluida su documentación (README y `docs/ARQUITECTURA.md`). La API firmará tokens con HS256 (configurable con `JWT_ALGORITHM`), con la misma interfaz `encode` / `decode`.

## Consecuencias

- Se eliminan `ecdsa`, `rsa`, `pyasn1` y `six` del árbol de dependencias.
- El control de seguridad del pipeline queda sin excepciones permanentes.
- Las variables `JWT_SECRET`, `JWT_ALGORITHM` y `JWT_EXPIRES_MINUTES` no cambian.

## Alternativas consideradas

Mantener `python-jose` e ignorar el aviso con `pip-audit --ignore-vuln`, justificado en que HS256 no usa `ecdsa`. Se descartó porque crea una excepción permanente cuya validez habría que re-verificar en cada cambio de algoritmo o de dependencias.
