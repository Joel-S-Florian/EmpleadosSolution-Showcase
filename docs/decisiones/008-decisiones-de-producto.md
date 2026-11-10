# ADR 008: Decisiones de producto tras la revisión de usabilidad (2026-11-09)

**Contexto.** Una revisión del sistema como usuario dejó seis decisiones que no eran técnicas sino de producto. Se registran con lo que se eligió, lo descartado y el estado.

| # | Decisión | Elegido | Descartado | Estado |
|---|---|---|---|---|
| 1 | Modelo de negocio | **SaaS multiempresa** (registro de empresas, usuarios por empresa, planes y límites, aislamiento reforzado) | Una sola empresa; multiempresa solo para despachos | Planificado: Fase 12 |
| 2 | Máximo de préstamo | **Cuota ≤ 30 % del sueldo del período** | Tope por monto (N × salario); ambos configurables | Hecho. Constante en `PrestamoService`, con `shortcut:` para hacerla configurable |
| 3 | Causas de terminación | **Mantener las causas legales y añadir ayuda** de qué paga cada una | Agruparlas bajo un nombre genérico | Hecho (ayuda desplegable en Liquidaciones) |
| 4 | Asistencia | **RRHH registra faltas** (como hoy); marcación del empleado queda como mejora futura | Marcación desde el portal | Sin cambios |
| 5 | Envío de recibos | **Interruptor por empresa, apagado por defecto**; envío manual siempre disponible | Solo manual | Hecho (`Empresa.EnvioAutomaticoRecibos`, migración). Antes se enviaba siempre al pagar |
| 6 | Respaldos | **Respaldo programado en el servidor + descarga solo Admin con registro** | Solo documentar | Planificado: Fase 13 |

**Por qué se registran.** Cada una cambia lo que el sistema promete. En entrevista: "No lo decidí por gusto técnico; lo puse como decisión de producto con alternativas, y lo barato lo hice de inmediato y lo caro lo planifiqué."

**Riesgos conocidos.**
- (2) El 30 % usa el sueldo bruto del período, no el neto, y no mira otras deudas. Para sueldo por hora no se valida.
- (5) Cambia el comportamiento anterior: las empresas existentes quedan con el envío automático apagado.
- (1) y (6) no están hechas: ver las fases 12 y 13 de [PLAN_MEJORAS.md](../PLAN_MEJORAS.md).
