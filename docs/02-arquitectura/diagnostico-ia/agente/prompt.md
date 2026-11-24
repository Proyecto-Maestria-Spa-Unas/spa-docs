# Prompt y ejecución del auditor

## Registro disponible

**Prompt histórico de lenguaje natural: sin evidencia disponible.** En el punto de entrada, el consolidador y la configuración Docker revisados no hay un prompt de modelo ni un parámetro para suministrarlo. El informe JSON tampoco registra un prompt. La orden reproducible de la ejecución fue `python -m agent`, que es un comando de entrada, no un prompt de lenguaje natural.

## Qué ejecutó el auditor

1. Validó las rutas locales de repositorios, trazabilidad y salida.
2. Inventarió los repositorios, buscó patrones y clasificó las coincidencias.
3. Extrajo y comparó las matrices documentales y los endpoints.
4. Construyó el diagnóstico arquitectónico, separando los hallazgos documentales de las inferencias y las alternativas con evidencia insuficiente.
5. Validó el informe `1.3.0` antes de escribir el JSON en `reports/`.

La ejecución registrada usó el motor `0.6.1.1` y el modo `solo_lectura`. Incluyó 10 repositorios, 301 archivos y 1.531 evidencias clasificadas. Los 13 hallazgos comprobados corresponden a la comparación de matrices. La evidencia arquitectónica quedó `parcial`, y los cuatro dominios examinados quedaron `no_evaluable`. El informe no acredita la implementación de una HU por su sola presencia documental ni justifica un piloto de extracción.

## Fuentes locales

Las rutas de esta sección son relativas a la raíz del proyecto externo `agente-auditor-ia`. El resultado académico se resume en el [diagnóstico arquitectónico](../diagnostico-arquitectura.md).

- `docker/Dockerfile` define el comando de entrada.
- `agent/__main__.py` valida rutas y coordina la ejecución.
- `agent/inspector/consolidador.py` consolida y valida la salida.
- `reports/diagnostico_arquitectura_20261123_232443_710295.json` registra el resultado observado.

Refs #46
