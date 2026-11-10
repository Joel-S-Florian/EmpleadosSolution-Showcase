# ADR 006: Observabilidad (logs, errores y salud)

**Contexto.** Casi no había logging (4 usos de `ILogger`), el log de desarrollo estaba lleno del SQL de cada consulta, la API no tenía manejo global de errores y no había forma de saber si la aplicación llegaba a la base de datos. Para depurar una nómina en producción había que reconstruir lo ocurrido a ciegas.

**Decisión.**
- **Logs estructurados** con plantillas (`{NominaId}`, `{EmpresaId}`…, no texto interpolado) en el ciclo de nómina (creada, calculada con conteo de recibos y de errores, aprobada, reabierta, pagada, cerrada, conflicto de concurrencia) y en las liquidaciones (calculada, pagada). Siguen la convención ya existente en `NotificacionService`: `ILogger<T>?` opcional, así las pruebas que construyen el servicio a mano no cambian.
- **Sin datos sensibles.** Se registran identificadores (nómina, empresa, empleado, usuario), nunca salarios, nombres ni correos. Una prueba lo verifica: corre un ciclo completo y comprueba que ni el salario ni el apellido aparecen en el log.
- **Nivel de EF Core a `Warning`** en `appsettings.json` de los dos hosts: el SQL de cada consulta ya no inunda el log (se puede reactivar con una variable de entorno al depurar).
- **Errores de la API:** `IExceptionHandler` (`ManejadorExcepciones`) + `ProblemDetails`. Registra la excepción con su traza y responde sin el mensaje interno. Un `DbUpdateConcurrencyException` es un `409 Conflict`, no un 500: cubre los cambios de nómina que aún no capturan el conflicto (ver ADR 005).
- **`/health`** en la API y el MVC con `BaseDeDatosHealthCheck` (comprueba que llega a la base). Sin paquetes nuevos.

**Alternativas descartadas.**
- Serilog u otro proveedor: `Microsoft.Extensions.Logging` alcanza y el destino (consola, archivo, nube) se decide al desplegar.
- Registrar el contenido de los recibos para "poder reconstruir" un cálculo: expone salarios. Lo reconstruible está en la base (cada recibo guarda su explicación paso a paso); el log solo dice qué pasó y cuándo.
- `AddDbContextCheck` del paquete de health checks de EF: añade una dependencia para algo que son 5 líneas.

**Consecuencias.** `CicloNominaService` y `LiquidacionService` aceptan un logger opcional. El MVC mantiene su página de error en producción; el manejador global es de la API.

**En entrevista (30 s).** "Para depurar una nómina en producción registro qué pasó, con qué identificadores y cuántos errores tuvo, nunca los montos. Los errores inesperados salen como ProblemDetails sin detalles internos y un conflicto de concurrencia es un 409, no un 500."
