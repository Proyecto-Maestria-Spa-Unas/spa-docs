# ADR-0010 · Broker de trabajos con priorización sobre PostgreSQL

- **Estado:** Propuesto
- **Fecha:** 2026-11-20
- **Decisores:** ivonnelizethmoreno-cloud (autora) · JJSUEspSIG (PO y QA)
- **Relacionado:** spa-backend-api#75 · spa-database#42 · spa-backend-api#76 · RN-09 · RN-10 · ADR-0003

## Contexto

Hasta ahora todo el sistema es síncrono: cada petición HTTP hace su trabajo completo antes de responder.
Algunas tareas no necesitan que el usuario las espere y tienen distinta urgencia. La primera es avisar
cuando un producto se agota (RN-10) o cruza su stock mínimo (RN-09).

Condiciones del entorno:

- La API corre en Vercel como funciones serverless: no hay procesos que queden escuchando una cola.
- La base es PostgreSQL de Supabase. El CI y el Docker local usan la imagen `postgis/postgis`, que no
  trae extensiones como `pgmq` (Supabase Queues).
- El movimiento es un libro mayor inmutable que se aplica con triggers en la misma transacción (ADR-0003).
  Un aviso de stock no debe existir si el movimiento que lo causó se deshizo.

## Decisión

Implementar un **broker de trabajos dentro del mismo PostgreSQL**, sin extensiones:

1. **Cola persistente** `broker.trabajo` con prioridad 1 alta, 2 normal y 3 baja.
2. **Consumo concurrente seguro** con `SELECT … FOR UPDATE SKIP LOCKED`: dos consumidores nunca toman el
   mismo trabajo y ninguno espera al otro.
3. **Envejecimiento:** cada 5 minutos de espera la prioridad efectiva sube un nivel, sin pasar de alta.
   Un trabajo de baja prioridad nunca espera indefinidamente.
4. **Reintentos con espera exponencial** (30 s, 60 s, 120 s… con tope de 1 h). Al agotar `max_intentos`
   el trabajo queda `FALLIDO`: es la cola de fallidos, consultable en `GET /broker/metricas`.
5. **Idempotencia:** `clave_idempotencia` única al encolar (`broker.encolar`) y una notificación por trabajo.
6. **Patrón outbox:** el trigger `trg_movimiento_encolar_alerta` encola `alerta_stock` en la misma
   transacción del movimiento. `AGOTADO` va con prioridad 1 y `BAJO_STOCK` con prioridad 2; este último
   solo cuando el saldo cruza el mínimo, no en cada salida posterior.
7. **Consumidor:** `POST /broker/procesar`, protegido con la cabecera `X-Broker-Token`. Lo invoca cada
   10 minutos el workflow `broker.yml` de GitHub Actions, igual que ya se hace con `keep-alive` y `respaldo`
   en spa-database. Cada trabajo se procesa en un punto de guardado: el fallo de uno no deshace los demás.
8. **Módulo hexagonal** `app/modulos/broker` (S5), con las mismas fronteras de import-linter que los
   otros servicios y su contrato `docs/openapi/broker-v1.yaml`.

## Consecuencias

**Positivas**

- Consistencia transaccional: la alerta nace y muere con el movimiento que la causó.
- Sin infraestructura nueva ni costos adicionales; funciona igual en Supabase, en el CI y en Docker.
- Priorización real, con protección contra inanición, reintentos y cola de fallidos.
- Si un módulo se extrae a su propio servicio, la cola ya separa productor y consumidor.

**Negativas y límites**

- La latencia del procesamiento depende del intervalo del workflow (10 minutos), y GitHub puede retrasar
  los workflows programados. No sirve para tareas que deban ejecutarse en segundos.
- Los workflows programados solo corren desde la rama por defecto (`main`). En staging (`develop`) la cola
  se procesa invocando el endpoint hasta el siguiente release.
- La cola comparte recursos con la base transaccional. Para volúmenes altos convendría un broker dedicado.
- Prioriza **trabajo asíncrono**. Priorizar peticiones HTTP en tiempo real (control de admisión bajo
  saturación) queda fuera de este ADR.
- Hay un secreto más que administrar: `BROKER_TOKEN` en Vercel y en GitHub.

## Alternativas consideradas

| Alternativa | Por qué no ahora |
|---|---|
| Supabase Queues (`pgmq`) | No está en la imagen `postgis/postgis` del CI ni en el Docker local: las migraciones fallarían en las pruebas |
| `LISTEN/NOTIFY` | No persiste mensajes: si no hay un proceso escuchando, el aviso se pierde. En serverless no hay escucha permanente |
| AWS SQS / EventBridge | Agrega cuenta, IAM, red y costos. Queda como evolución en spa-infra-aws si el volumen lo justifica |
| RabbitMQ o Kafka | Requieren servidores administrados permanentes; desproporcionados para el volumen de un spa |
| Redis con Celery o RQ | Necesitan workers vivos, que Vercel no ofrece, y otra pieza de infraestructura |
| Cron de Vercel | Sus límites de frecuencia dependen del plan (no verificado); GitHub Actions ya se usa en el proyecto |
