# ADR 003: Las reglas de estado del empleado viven en la entidad

**Contexto.** La regla "un empleado despedido no se modifica" estaba escrita a mano en cuatro lugares de `EmpleadoService` y el estado se asignaba directamente en `LiquidacionService`. Cualquier servicio nuevo podía olvidar la regla o cambiar `Estado` sin pasar por ella.

**Decisión.** `Empleado.Estado` y `Empleado.FechaTerminacion` pasan a `private set`. Las transiciones son métodos de la entidad que devuelven `Resultado`: `Desactivar()`, `Despedir(fecha)`, `FijarFechaTerminacion(fecha)` y `ComprobarModificable()`. Los servicios orquestan (cargar, llamar al método, registrar el movimiento, guardar) y ya no contienen la regla. EF Core escribe los setters privados sin configuración extra, y `has-pending-model-changes` sigue limpio.

**Alcance deliberado.** Solo `Empleado`, que tiene invariantes reales (Despedido es estado final). No se rediseñó el resto como DDD. No se añadió `Reactivar()`: ningún flujo lo usa todavía.

**Alternativas descartadas.**
- Dejar el setter público y confiar en revisión de código: la regla sigue siendo saltable.
- `init`: impide también que los métodos de la entidad cambien el estado.
- Excepciones de dominio: el proyecto ya usa `Resultado` para reglas de negocio, así que se mantiene el mismo estilo.

**Consecuencias.** Datos de ejemplo y pruebas ya no asignan `Estado` en inicializadores; construyen el objeto y llaman a `Desactivar()`/`Despedir()`. Hay pruebas de la entidad sin base de datos (`EmpleadoDominioTests`).

**En entrevista (20 s).** "La regla está dentro de la entidad: no hay forma de saltársela desde otro servicio. Lo apliqué donde había una invariante real, no a todo el modelo."
