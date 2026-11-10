# ADR 005: Pruebas de integración contra SQL Server real

**Contexto.** Las 170+ pruebas usan EF InMemory, que no es relacional: no aplica migraciones, no hace cumplir índices únicos ni claves foráneas, no traduce el SQL real y no tiene concurrencia verdadera. Daba confianza sobre la lógica, pero no sobre lo que depende del motor.

**Decisión.** Un grupo pequeño de pruebas (`Categoria=Integracion`, carpeta `Empleados.Tests/SqlServer`) corre contra SQL Server real. Cada clase de pruebas crea una base propia con las migraciones (`MigrateAsync`) y la borra al terminar. Se activan con la variable `EMPLEADOS_TEST_SQL` (cadena de conexión a la instancia); sin ella se omiten, no fallan. En CI hay un job aparte con un contenedor de SQL Server como `service`.

Cubren: migraciones desde cero sin pendientes, el filtro por día contra SQL real, índice único de correo por empresa (y que otra empresa sí pueda repetirlo), clave foránea `Restrict` al borrar una empresa, y **concurrencia al pagar una nómina**.

**Hallazgo.** La prueba de concurrencia falló al escribirla: dos sesiones que leían la nómina "Aprobada" la pagaban las dos con éxito (no había control de concurrencia). Corrección: `NominaPeriodo.RowVersion` (`[Timestamp]`, migración `NominaRowVersion`) y `GuardarTransicionAsync`, que convierte `DbUpdateConcurrencyException` en un `Resultado.Invalido` en aprobar, reabrir, pagar y cerrar. Como el pago de préstamos y el cambio de estado se guardan juntos, el conflicto revierte ambos.

**Alternativas descartadas.**
- Testcontainers: autocontenido, pero exige Docker en cada máquina (aquí no está instalado) y un paquete más. La variable de entorno funciona igual en local (cualquier instancia) y en CI (contenedor como `service`), con un solo camino de código.
- SQLite en memoria: relacional, pero con otro dialecto; no valida lo específico de SQL Server (rowversion, índices filtrados).
- Pasar todas las pruebas a SQL Server: más lento y sin necesidad; InMemory sigue sirviendo para la lógica.

**Consecuencias.** Un job más en CI. `NominaPeriodo` gana una columna `rowversion`. Los demás cambios de la nómina (recalcular, editar novedades) no capturan el conflicto todavía: lanzarían la excepción. Queda anotado como siguiente paso si se observa en uso.

**En entrevista (30 s).** "InMemory no es relacional, así que añadí pruebas contra SQL Server real para migraciones, índices y concurrencia. La de concurrencia encontró un bug real: dos usuarios podían pagar la misma nómina. Lo resolví con un token de concurrencia y un mensaje claro."
