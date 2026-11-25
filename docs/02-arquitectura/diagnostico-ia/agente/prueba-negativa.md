# Protocolo de pruebas negativas de escritura en GitHub

**Estado: PENDIENTE DE EJECUCIÓN.** Este documento prepara el protocolo y el registro. No hay prueba positiva documentada ni resultados de `git push` o `gh issue comment`. HTTP 403 es el resultado esperado, pendiente de observación.

## Autorizaciones e insumos

El administrador debe designar un repositorio privado desechable, fuera de los diez repositorios auditados. Debe autorizar su propietario, una rama o referencia descartable, un issue abierto para pruebas, la identidad de prueba, la credencial temporal y la limpieza si una operación llega a escribir. El PAT fine-grained de prueba debe estar limitado a ese repositorio, con permisos de lectura y vigencia máxima de siete días.

Realizar las pruebas desde un entorno separado del contenedor auditor. Usar transporte HTTPS con la identidad restringida, sin reutilizar autenticación persistente del equipo. Acordar el texto inocuo del comentario y una actualización Git real, no vacía y limitada a la referencia de prueba. Mantener deshabilitada la publicación de registros crudos. No registrar tokens, cabeceras de autenticación ni trazas sensibles.

## Protocolo propuesto

1. **Aislar y comprobar alcance.** Confirmar que el remoto, la referencia y el issue pertenecen al entorno desechable y que el cliente solo puede usar la identidad autorizada. Confirmar vigencia y alcance efectivo sin revelar secretos.
2. **Realizar el control positivo con la misma identidad restringida.** Leer repositorio, referencia e issue. Conservar el identificador inicial de la referencia y el estado inicial de comentarios. Si falla la lectura, detenerse y clasificar la prueba como inconclusa.
3. **Probar `git push`.** Intentar una escritura real por HTTPS sobre la referencia descartable. Un push sin cambios o en modo simulación no demuestra denegación. Registrar la evidencia del HTTP 403 observado y el código de salida. Volver a leer la referencia para comprobar que no cambió.
4. **Probar `gh issue comment`.** Intentar publicar el comentario acordado en el issue desechable. Registrar la evidencia del HTTP 403 observado y el código de salida. Volver a leer los comentarios para comprobar que la prueba no añadió ninguno.
5. **Clasificar y cerrar.** Cada negativa se acredita con lectura positiva previa, HTTP 403 atribuible a los permisos y ausencia de cambio. Un `401`, `404`, fallo de red o rechazo atribuible a reglas de rama o al estado del issue es inconcluso. Si alguna escritura tiene éxito, detenerse, registrar el fallo del control y remitir la limpieza al responsable autorizado.

## Registro por completar

| Campo | Valor |
|---|---|
| Aprobación, responsable y fecha | Pendiente |
| Propietario y repositorio de prueba | Pendiente |
| Referencia Git e issue autorizados | Pendiente |
| Identidad de prueba, alcance efectivo, fecha de emisión y caducidad | Pendiente. Registrar únicamente metadatos no secretos. |
| Herramientas y versiones | Pendiente |
| Control positivo de lectura | Pendiente |
| Identificador inicial y final de la referencia | Pendiente |
| Estado inicial y final de los comentarios | Pendiente |
| Limpieza, revocación o caducidad de la credencial | Pendiente |

| Prueba | Fecha y recurso | Código de salida | HTTP observado | Estado posterior | Evidencia saneada y conclusión |
|---|---|---|---|---|---|
| `git push` | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |
| `gh issue comment` | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |

Fuente del proyecto externo `agente-auditor-ia`: `config/POLITICA-SEGURIDAD.md`, sección de pruebas negativas de permisos GitHub. La inspección del contenedor y la ejecución local prueban controles distintos de estas denegaciones remotas.

Refs #46
