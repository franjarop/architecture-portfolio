# ADR-002 — Adopción de CQRS con MediatR en servicios de fraude

| Campo       | Valor                                      |
|-------------|--------------------------------------------|
| **Estado**  | ✅ Aceptado                                |
| **Fecha**   | 2025-05-16                                 |
| **Autor**   | Francisco Javier Robles Pérez              |
| **Revisores** | Tech Lead · Architecture team            |
| **Reemplaza** | N/A                                      |

---

## 1. Contexto

El servicio de prevención de fraude maneja dos tipos de operaciones con
naturaleza muy diferente: consultas de estado de transacciones (lectura) y
comandos de bloqueo, reverso o aprobación (escritura). Ambas tenían
responsabilidades mezcladas en los mismos controladores y servicios de
aplicación, lo que generaba:

- Clases de servicio con más de 500 líneas difíciles de mantener
- Tests unitarios complejos por el alto acoplamiento entre lectura y escritura
- Dificultad para escalar lecturas independientemente de las escrituras
- Cambios en la lógica de consulta afectaban inadvertidamente flujos de comando

El equipo tiene experiencia previa con el patrón Repository tradicional pero
el crecimiento del dominio de fraude hacía insostenible mantener ese enfoque.

---

## 2. Decisión

Se adopta **CQRS (Command Query Responsibility Segregation) con MediatR**
como patrón de organización de la capa de aplicación en los servicios de
prevención de fraude, separando explícitamente comandos de consultas mediante
handlers independientes.

---

## 3. Opciones evaluadas

### Opción A — Servicios de aplicación tradicionales
- **Pro:** Familiar para el equipo, curva de aprendizaje baja
- **Pro:** Menos archivos y clases en el proyecto
- **Con:** Mezcla de responsabilidades de lectura y escritura
- **Con:** Difícil de escalar o optimizar de forma independiente
- **Con:** Tests más complejos por el acoplamiento

### Opción B — CQRS con MediatR ✅ *(elegida)*
- **Pro:** Separación clara: un handler = una responsabilidad
- **Pro:** Pipeline behaviors de MediatR permiten agregar logging,
  validación y transacciones de forma transversal sin tocar los handlers
- **Pro:** Tests unitarios simples: cada handler se prueba de forma aislada
- **Pro:** Facilita evolución hacia read models optimizados en el futuro
- **Con:** Mayor número de clases y archivos en el proyecto
- **Con:** Curva de aprendizaje inicial para developers nuevos en el equipo

### Opción C — Clean Architecture sin CQRS (solo use cases)
- **Pro:** Estructura más simple, menos abstracciones
- **Con:** Sin separación explícita de lectura/escritura, el problema persiste
- **Con:** No aprovecha el pipeline de MediatR para cross-cutting concerns

---

## 4. Justificación

El dominio de fraude tiene consultas de alto volumen (verificación en tiempo
real por cada transacción) y comandos de baja frecuencia pero alto impacto
(bloqueos, reversos). CQRS permite optimizar cada lado de forma independiente
sin que los cambios interfieran entre sí.

MediatR en particular resuelve el problema de cross-cutting concerns: logging
de auditoría, validación con FluentValidation y manejo de transacciones se
implementan una sola vez como pipeline behaviors y aplican automáticamente
a todos los comandos, sin repetir código en cada handler.

La decisión está alineada con la dirección técnica de Microsoft para aplicaciones
.NET de mediana y alta complejidad.

---

## 5. Consecuencias

### Positivas
- Cada comando y query tiene un único handler con responsabilidad clara
- El pipeline de MediatR centraliza logging, validación y transacciones
- Los tests unitarios de handlers son simples y rápidos de escribir
- Es posible en el futuro separar la base de datos de lectura de la de escritura
  sin cambiar la capa de aplicación

### Negativas / Trade-offs aceptados
- El número de archivos del proyecto aumenta significativamente
- Los developers nuevos necesitan entender el patrón antes de contribuir
- El overhead de MediatR es mínimo pero existe (~microsegundos por request)

### Acciones requeridas
- [ ] Crear pipeline behavior para logging de auditoría de comandos
- [ ] Crear pipeline behavior para validación automática con FluentValidation
- [ ] Documentar la convención de nomenclatura: `VerbNounCommand`, `VerbNounQuery`
- [ ] Agregar ejemplos de handler en el README del proyecto

---

## 6. Links y referencias

- **Librería:** [MediatR — GitHub](https://github.com/jbogard/MediatR)
- **Patrón:** [CQRS — Martin Fowler](https://martinfowler.com/bliki/CQRS.html)
- **Docs Microsoft:** [CQRS pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)
- **ADR relacionado:** [ADR-003 — Clean Architecture como base estructural](./ADR-003-clean-architecture.md)

---

## 7. Historial de cambios

| Fecha      | Autor                          | Cambio              |
|------------|--------------------------------|---------------------|
| 2025-05-16 | Francisco Javier Robles Pérez  | Creación del ADR    |
| 2025-05-16 | Francisco Javier Robles Pérez  | Estado: Aceptado    |
