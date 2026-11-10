# Flujos y preguntas de diseño (Fase 11, en curso)

Documento vivo: responde dudas sobre qué hace cada módulo, registra lo ya cambiado y lo que falta por decidir. Nació de una revisión del sistema el 2026-11-09.

## Mapa mental: cinco áreas, no veinte módulos

| Área | Módulos que hoy viven aquí |
|---|---|
| **Empleados** | Empleados, Departamentos, Puestos, Incidencias y vacaciones, Préstamos, Liquidaciones |
| **Nóminas** | Nóminas, Navidad y bonificaciones, Simulador |
| **Pagos y archivos** | Pago al banco, archivo TSS, planilla de personal |
| **Reportes** | Reportes (PDF, Excel, CSV) |
| **Configuración** | Parámetros legales, Empresas, Notificaciones |

Pendiente: reorganizar el menú en estas cinco áreas (sin mover código; es solo navegación).

## Respuestas

**¿Por qué hay nóminas quincenales y mensuales?** En República Dominicana cada empleado se paga con la frecuencia pactada: quincenal o mensual. Una empresa puede tener ambos grupos. Cada nómina incluye solo a los empleados de su frecuencia. Una misma empresa puede tener, para el mismo mes, una mensual y dos quincenales (cada una con sus empleados).

**¿Se pueden crear dos nóminas del mismo mes?** No del mismo período: el sistema rechaza una segunda con igual empresa, año, mes, frecuencia y quincena ("Ya existe una nómina para ese período"). Sí pueden coexistir frecuencias distintas.

**¿Qué es "Δ vs anterior"?** La diferencia del neto de ese empleado contra su recibo en la nómina anterior. No es contable: sirve para detectar errores (un neto que cae a la mitad suele ser una novedad mal cargada). Ya tiene una explicación al pasar el cursor.

**¿Funciona "Enviar recibos por correo"?** Sí, con un servidor SMTP en la sección `Correo` de `appsettings` (`OpcionesCorreo`). **Sin `Host` configurado no se envía nada: los correos se guardan como archivos .eml en una carpeta de prueba** (pensado para desarrollo). Un fallo de envío real queda registrado como "fallido". Cada envío queda guardado (a quién, cuándo, resultado), así que reenviar es seguro y se ve cuántos van enviados. Ya se muestra arriba, junto al botón.

**¿Navidad y bonificaciones?** Son dos pagos legales que no son el sueldo mensual:
- *Salario de Navidad* (regalía pascual): un doceavo de lo ganado en el año, se paga en diciembre. Crea una nómina especial.
- *Bonificación*: participación en las utilidades de la empresa (el 10% de los beneficios, con topes por antigüedad). Se reparte a los empleados con derecho.
Hoy están juntos porque ambos son "nóminas especiales"; la pantalla necesita explicarlo mejor.

**¿Archivos TSS y banco?**
- *Archivo TSS*: la TSS exige cada mes un archivo con lo que se descontó y aportó por cada empleado (AFP, SFS, etc.). Se sube a su plataforma para pagar la seguridad social.
- *Pago al banco*: archivo para que el banco deposite los netos de la nómina de una vez, en lugar de transferir uno por uno.
- *Planilla de personal*: la base del formulario DGT-3 del Ministerio de Trabajo (listado oficial del personal de la empresa). Es distinta del reporte "Plantilla de empleados" (ese es interno).

**¿Contratos por vencer?** Existen contratos por tiempo definido u obra (campo `FechaFinContrato`). Los indefinidos no aparecen. Si no se usan, el reporte sale vacío.

**¿Rotación de personal?** Por mes: empleados al inicio, altas, bajas y al final, y el porcentaje de rotación (bajas ÷ promedio de empleados). Mide cuánta gente entra y sale.

**¿Empresas? ¿Es un SaaS?** Hoy el sistema es multiempresa (cada empresa ve solo sus datos, probado con `MultiempresaTests`). Eso es una decisión de negocio, no técnica: ver "Por decidir".

## Ya cambiado

- Meses con mayúscula inicial (selectores, reportes y encabezados).
- Nóminas: el mes es un selector y la quincena se deshabilita cuando la frecuencia es mensual.
- Nóminas: el resumen de recibos enviados está arriba, junto al botón.
- Préstamos: se indica el **número de cuotas** y la cuota se calcula (monto ÷ cuotas, redondeada hacia arriba). La API acepta todavía `MontoCuota` directo.

## Decisiones tomadas

Resueltas el 2026-11-09 y registradas en el [ADR 008](decisiones/008-decisiones-de-producto.md): SaaS multiempresa (Fase 12), tope de cuota 30 % (hecho), causas legales con ayuda (hecho), asistencia sin cambios, envío automático de recibos por empresa (hecho), respaldos (Fase 13).

## Cola de trabajo (en orden sugerido)

| # | Tema | Tamaño |
|---|---|---|
| 1 | Búsqueda, filtros y paginación en Nóminas, Liquidaciones, Departamentos, Puestos, Incidencias (Fase 9) | M |
| 2 | Departamento: detalle con sus empleados | S |
| 3 | Puestos: más datos (nivel, rango salarial, requisitos) | S |
| 4 | Incidencias: pantalla de consulta global con filtros; vacaciones colectivas más claras | M |
| 5 | Notificaciones: estilo bitácora, búsqueda, leída individual y en bloque (Fase 10) | M |
| 6 | Respaldos recurrentes y descarga de la base de datos | M, riesgo de seguridad: la descarga expone todo |
| 7 | Reordenar el menú en las 5 áreas; textos de ayuda en Navidad/Bonificación y Archivos | S |
| 8 | Simulador: prueba que lo compare con una nómina real para que no se desactualice | S |
