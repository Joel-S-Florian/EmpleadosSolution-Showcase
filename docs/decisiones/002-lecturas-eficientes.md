# ADR 002: Lecturas eficientes en el listado de empleados

**Contexto.** `ListarAsync` cargaba la entidad completa con tres `Include`, con seguimiento de cambios, aunque solo se leía. El filtro `FechaContratacion.Date == x` impedía usar índices.

**Decisión.**
- La API usa `ListarDtoAsync`: `.Select(EmpleadoDto.Proyeccion)` trae solo las columnas del DTO. La proyección es una `Expression` única; `EmpleadoDto.Desde` la compila, así no hay dos mapeos que mantener.
- El MVC conserva `ListarAsync` con entidades (sus vistas leen la entidad), ahora con `AsNoTracking`.
- Filtro por día como rango `>= dia AND < dia+1`.
- `AsNoTracking` en detalle y lecturas del generador de reportes (ninguna escribe).

**Alternativas descartadas.** Cambiar el MVC a ViewModels: más cambios sin beneficio medido. `AsNoTracking` global en el `DbContext`: rompería los servicios que sí modifican entidades.

**Verificación.** Pruebas del filtro por día y de que el listado no deja entidades seguidas. Contra SQL Server real se comprobó el `SELECT` con joins y el `WHERE` por rango (log de EF).

**En entrevista (20 s).** "Miré el SQL generado: en lecturas pido solo lo que necesito y sin seguimiento; el filtro por fecha es un rango para poder usar índice."
