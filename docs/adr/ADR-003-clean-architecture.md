# ADR-003 — Clean Architecture como base estructural de los microservicios

| Campo       | Valor                                      |
|-------------|--------------------------------------------|
| **Estado**  | ✅ Aceptado                                |
| **Fecha**   | 2025-05-16                                 |
| **Autor**   | Francisco Javier Robles Pérez              |
| **Revisores** | Tech Lead · Architecture team            |
| **Reemplaza** | N/A                                      |

---

## 1. Contexto

Los microservicios del sistema de prevención de fraude crecieron sin una
estructura de proyecto consistente. Cada servicio organizaba sus capas de
forma diferente, lo que generaba:

- Alta fricción al incorporar developers nuevos al proyecto
- Lógica de negocio acoplada directamente a Entity Framework y controladores
- Tests de integración lentos porque la lógica dependía de la base de datos
- Dificultad para cambiar detalles de infraestructura (proveedor de BD,
  librería HTTP) sin modificar la lógica de dominio

Se evaluó adoptar una arquitectura consistente para todos los servicios nuevos
y como guía de refactorización de los existentes.

---

## 2. Decisión

Se adopta **Clean Architecture** (Robert C. Martin) como estructura base de
los microservicios, organizando el código en cuatro capas con dependencias
que apuntan siempre hacia el dominio: Domain, Application, Infrastructure
y Presentation.

---

## 3. Opciones evaluadas

### Opción A — Arquitectura en capas tradicional (N-Layer)
- **Pro:** Familiar para la mayoría del equipo
- **Pro:** Menos abstracciones, estructura más plana
- **Con:** Las capas superiores dependen de las inferiores, acoplando
  la lógica de negocio a la infraestructura
- **Con:** Tests unitarios requieren mocks de base de datos

### Opción B — Clean Architecture ✅ *(elegida)*
- **Pro:** La lógica de dominio no depende de ningún framework ni librería externa
- **Pro:** Tests unitarios de la capa Application son rápidos y sin dependencias de BD
- **Pro:** Es posible cambiar Entity Framework por otro ORM sin tocar el dominio
- **Pro:** Estructura predecible: cualquier developer sabe dónde buscar cada cosa
- **Con:** Mayor número de proyectos y abstracciones (interfaces de repositorio)
- **Con:** Puede sentirse como over-engineering en servicios muy simples

### Opción C — Vertical Slice Architecture
- **Pro:** Organización por feature en lugar de por capa
- **Pro:** Menos acoplamiento entre features distintas
- **Con:** Menos familiar para el equipo actual
- **Con:** Requiere disciplina alta para evitar duplicación entre slices
- **Con:** Difícil de aplicar retroactivamente a servicios existentes

---

## 4. Justificación

El dominio de fraude contiene reglas de negocio complejas que deben poder
probarse sin levantar base de datos ni servicios externos. Clean Architecture
lo garantiza al mantener la capa Domain y Application completamente
independientes de Entity Framework, Azure Service Bus y cualquier otro
detalle de infraestructura.

La regla de dependencia (las capas externas dependen de las internas, nunca
al revés) es la que permite sustituir o actualizar proveedores de
infraestructura sin riesgo de romper la lógica de negocio.

Vertical Slice fue considerada pero descartada porque la mayor parte del
equipo no tiene experiencia con ese enfoque y la migración de servicios
existentes sería más costosa.

---

## 5. Consecuencias

### Positivas
- La lógica de negocio en Domain y Application es testeable sin dependencias externas
- Todos los microservicios nuevos siguen la misma estructura, reduciendo fricción
- Es posible reemplazar Entity Framework o cambiar de SQL Server a otro motor
  sin modificar las capas internas
- Onboarding de developers nuevos es más rápido con estructura predecible

### Negativas / Trade-offs aceptados
- Cada microservicio tiene al menos 4 proyectos (.Domain, .Application,
  .Infrastructure, .API), lo que incrementa la complejidad inicial
- Las interfaces de repositorio en Application generan indirección que
  puede parecer innecesaria en servicios simples
- Requiere disciplina del equipo para no crear dependencias hacia afuera
  desde las capas internas

### Acciones requeridas
- [ ] Crear template de proyecto con la estructura base de 4 capas
- [ ] Documentar la convención de carpetas dentro de cada capa
- [ ] Agregar architectural tests con NetArchTest para validar que
      las dependencias entre capas se respetan en CI/CD

---

## 6. Links y referencias

- **Libro:** Clean Architecture — Robert C. Martin (Uncle Bob)
- **Docs:** [Clean Architecture with .NET — Microsoft](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)
- **Template de referencia:** [ardalis/CleanArchitecture](https://github.com/ardalis/CleanArchitecture)
- **ADR relacionado:** [ADR-002 — CQRS con MediatR](./ADR-002-cqrs-mediatr.md)

---

## 7. Historial de cambios

| Fecha      | Autor                          | Cambio              |
|------------|--------------------------------|---------------------|
| 2025-05-16 | Francisco Javier Robles Pérez  | Creación del ADR    |
| 2025-05-16 | Francisco Javier Robles Pérez  | Estado: Aceptado    |
