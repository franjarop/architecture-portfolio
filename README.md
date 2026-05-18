# Architecture Portfolio — Francisco Javier Robles Pérez

Documentación técnica de decisiones arquitectónicas tomadas en sistemas
de pagos y prevención de fraude. Este repositorio refleja mi proceso de
razonamiento como arquitecto de software, no código propietario.

## Stack principal
- .NET 8 · Clean Architecture · CQRS + MediatR
- Azure (Service Bus, SQL Server, App Insights, API Management)
- Domain-Driven Design · Event-driven architecture

## Contenido

| Carpeta | Descripción |
|--------|-------------|
| `docs/adr/` | Architecture Decision Records |

## ADRs documentados

| ID | Título | Estado |
|----|--------|--------|
| [ADR-001](docs/adr/ADR-001-azure-service-bus.md) | Uso de Azure Service Bus para transacciones financieras | ✅ Aceptado |
| [ADR-002](docs/adr/ADR-002-cqrs-mediatr.md) | Adopción de CQRS con MediatR en servicios de fraude | ✅ Aceptado |
| [ADR-003](docs/adr/ADR-003-clean-architecture.md) | Clean Architecture como base estructural | ✅ Aceptado |
