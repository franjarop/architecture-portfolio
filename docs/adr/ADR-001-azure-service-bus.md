# ADR-001 — Uso de Azure Service Bus para procesamiento de transacciones financieras

| Campo       | Valor                                      |
|-------------|--------------------------------------------|
| **Estado**  | ✅ Aceptado                                |
| **Fecha**   | 2025-05-16                                 |
| **Autor**   | Francisco Javier Robles Pérez              |
| **Revisores** | Tech Lead · Architecture team            |
| **Reemplaza** | N/A                                      |

---

## 1. Contexto

El servicio de prevención de fraude necesita procesar transacciones financieras
aprobadas y solicitudes de reverso de forma desacoplada entre microservicios.

El volumen es de aproximadamente 12,000 transacciones/día con picos de hasta
800 transacciones por minuto en horario comercial. Las transacciones de reverso
tienen criticidad alta: un mensaje perdido representa pérdida económica directa
para el comercio afectado.

El estado anterior usaba llamadas HTTP síncronas directas entre servicios, lo
que generaba fallos en cascada cuando el servicio receptor tenía downtime.

---

## 2. Decisión

Se adopta **Azure Service Bus con modelo de topics y subscriptions** como bus de
mensajería para el flujo de transacciones entre los microservicios de aprobación
y prevención de fraude, en lugar de llamadas HTTP directas.

---

## 3. Opciones evaluadas

### Opción A — HTTP directo (REST síncrono)
- **Pro:** Simple de implementar, respuesta inmediata
- **Pro:** Sin infraestructura adicional
- **Con:** Acoplamiento fuerte entre servicios
- **Con:** Sin mecanismo de retry automático
- **Con:** Fallo en cascada si el servicio receptor cae

### Opción B — Azure Service Bus con topics ✅ *(elegida)*
- **Pro:** Desacoplamiento total entre emisor y receptor
- **Pro:** Dead-letter queue nativa para mensajes fallidos
- **Pro:** Soporte para múltiples subscriptores sin modificar el emisor
- **Pro:** At-least-once delivery garantizado
- **Con:** Latencia adicional de ~50ms vs HTTP directo
- **Con:** Costo mensual adicional (~$18 USD/mes en tier Standard)

### Opción C — RabbitMQ self-hosted
- **Pro:** Costo bajo, control total del broker
- **Con:** Requiere operación y mantenimiento adicional del equipo
- **Con:** Sin SLA gestionado por el proveedor
- **Con:** No alineado con el stack Azure existente del proyecto

---

## 4. Justificación

Azure Service Bus garantiza entrega **at-least-once** con dead-letter automática,
lo cual es no negociable para transacciones de reverso. Un mensaje perdido puede
representar cientos de dólares en disputas con el comercio.

El desacoplamiento permite que el servicio de fraude escale o se redespliegue
independientemente sin afectar el servicio de aprobación. El costo adicional de
~$18/mes está ampliamente justificado dado el costo de una sola transacción de
reverso perdida.

Adicionalmente, el stack ya opera sobre Azure, por lo que la operación y el
monitoreo quedan dentro del mismo ecosistema (Application Insights, Azure Monitor).

---

## 5. Consecuencias

### Positivas
- El servicio de fraude puede escalar o actualizarse sin coordinar con el servicio de aprobación
- Nuevos subscriptores (auditoría, analytics, notificaciones) pueden agregarse sin tocar el emisor
- Los mensajes fallidos quedan en dead-letter queue para revisión y reintento manual

### Negativas / Trade-offs aceptados
- El consumidor debe implementar **idempotencia**: un mismo mensaje puede llegar más de una vez
- El `MessageId` de cada transacción debe almacenarse para detectar y descartar duplicados
- La latencia de procesamiento deja de ser síncrona: el emisor no confirma procesamiento exitoso

### Acciones requeridas
- [ ] Implementar almacenamiento de `MessageId` procesados en la base de datos
- [ ] Configurar alertas en Azure Monitor para mensajes en dead-letter queue
- [ ] Documentar el contrato de mensaje (schema) del topic `approved-financial-transactions`

---

## 6. Links y referencias

- **Topic:** `approved-financial-transactions` → subscription: `transaction-to-reverse`
- **Servicio:** `fraud-prevention-db-sl-devpp`
- **Docs:** [Azure Service Bus overview](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)
- **Patrón relacionado:** [Competing Consumers Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/competing-consumers)

---

## 7. Historial de cambios

| Fecha      | Autor                          | Cambio              |
|------------|--------------------------------|---------------------|
| 2025-05-16 | Francisco Javier Robles Pérez  | Creación del ADR    |
| 2025-05-16 | Francisco Javier Robles Pérez  | Estado: Aceptado    |
