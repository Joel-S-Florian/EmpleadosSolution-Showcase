# ADR 001: Reloj inyectable (`TimeProvider`)

**Contexto.** Los servicios leían `DateTime.Now/Today` directamente (~90 usos). Fechas de nómina, vencimientos y recordatorios dependían del reloj del servidor y no se podían fijar en pruebas.

**Decisión.** Inyectar `System.TimeProvider` (ya incluido en .NET 8+). Dos extensiones, `Ahora()` y `Hoy()`, mantienen la semántica de hora local de `DateTime.Now/Today`. Se registra `TimeProvider.System` en DI; las pruebas usan `FakeTimeProvider`.

**Alternativas descartadas.**
- Interfaz propia `IReloj`: duplica algo que la plataforma ya ofrece.
- Reloj estático ambiental: es estado global; las pruebas en paralelo se pisan.
- Parámetro opcional con valor por defecto: se puede olvidar y volver al reloj real sin que nada falle.

**Consecuencias.** Constructores con un parámetro más (lo cubre DI). Quedan fuera, a propósito: pies de página de PDF/Excel, valores por defecto de DTO y vistas, porque solo muestran la hora de impresión y no afectan cálculos.

**En entrevista (20 s).** "El motor de nómina ya era puro; extendí la idea: nada del negocio lee el reloj directamente, así las fechas se prueban de forma determinista."
