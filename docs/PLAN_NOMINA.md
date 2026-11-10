# Plan: de "Gestión de Empleados" a Sistema de Nómina y RRHH (República Dominicana)

> Estado: **borrador para decidir**. Las cifras legales vienen de fuentes secundarias (blogs y guías 2026) y **deben validarse** con el texto oficial (DGII, TSS, Ministerio de Trabajo) y con un contador o abogado laboral antes de usarse con nóminas reales. Por eso el diseño las guarda como **parámetros con fecha de vigencia**, nunca escritas en el código.

---

## 1. Marco legal que el sistema debe cubrir

### 1.1 Seguridad Social (TSS, Ley 87-01)
| Concepto | Empleado | Empleador | Tope de salario cotizable 2026 |
|---|---|---|---|
| AFP (pensiones) | 2.87 % | 7.10 % | 20 salarios mínimos ≈ RD$464,460/mes |
| SFS (salud) | 3.04 % | 7.09 % | 10 salarios mínimos ≈ RD$232,230/mes |
| SRL (riesgos laborales) | — | ≈1.10 %–1.30 % (según riesgo de la empresa) | 4 salarios mínimos ≈ RD$92,892/mes |
| INFOTEP | — (0.5 % de la bonificación) | 1 % de la nómina | — |

- Pendiente de verificar: una fuente menciona un "Fondo de Solidaridad Social" de 0.40 % dentro del 7.10 % patronal. Además, los dependientes adicionales del SFS tienen un costo per cápita que se descuenta al empleado.
- Se reporta y paga mensualmente por el SUIR: archivo de **novedades** (altas, bajas, cambios) y archivo de **autodeterminación** (la nómina del mes). El layout está en el instructivo TSS v24.

### 1.2 ISR de asalariados (DGII)
- Escala anual 2026 (igual que la de 2025):
  - exento hasta RD$416,220;
  - 15 % del excedente hasta RD$624,329;
  - RD$31,216 + 20 % del excedente hasta RD$867,123;
  - RD$79,776 + 25 % del excedente.
- La base gravable es el salario menos los aportes del empleado a AFP y SFS.
- La retención mensual se calcula anualizando la base (×12), aplicando la escala y dividiendo entre 12.
- El IR-3 se genera desde lo que se envía a la TSS; el IR-17 es para otras retenciones.
- **No gravan ISR:** el Salario de Navidad (Regalía Pascual), el preaviso y la cesantía (art. 86), aunque sobre esto las fuentes no coinciden. Las vacaciones sí gravan. **Hay que validar cada caso.**
- Una fuente menciona una reforma futura (exención de RD$480,000 y un tramo del 27 % "desde 2027"). Como las escalas son versionadas por vigencia, adoptarla sería solo cargar una escala nueva.

### 1.3 Salario mínimo (CNS-01-2025, vigente desde el 1-feb-2026, sector no sectorizado)
| Empresa | Mensual |
|---|---|
| Grande | RD$29,988.00 |
| Mediana | RD$27,489.60 |
| Pequeña | RD$18,421.20 |
| Micro | RD$16,993.20 |

Otros sectores (zonas francas, hoteles, construcción, vigilantes, agrícola) tienen sus propias tarifas. El sistema debe **validar que ningún salario quede por debajo del mínimo** que le corresponde a la empresa.

### 1.4 Código de Trabajo (Ley 16-92)
| Tema | Regla |
|---|---|
| Salario diario | Salario mensual ÷ **23.83** (días laborables promedio); por hora: ÷ 8 |
| Jornada | 44 h semanales (8 h diarias) |
| Horas extra | +35 % sobre la jornada ordinaria hasta 68 h/semana; **+100 %** a partir de ahí (art. 203) |
| Nocturno (9 p.m.–7 a.m.) | Recargo mínimo de 15 % (art. 204) |
| Trabajo en día feriado o descanso semanal | +100 %, o día de descanso compensatorio |
| Vacaciones (art. 177/180) | 14 días laborables del año 1 al 5; 18 días después de 5 años; proporcionales en el primer año |
| Salario de Navidad (art. 219) | 1/12 de lo devengado en el año; se paga antes del 20 de diciembre; no cotiza TSS |
| Bonificación (art. 223) | 10 % de las utilidades netas, con tope de 45 días de salario (menos de 3 años) o 60 días (3 años o más); se paga entre 90 y 120 días después del cierre fiscal |
| Preaviso (art. 76) | 7 días (3 a 6 meses) · 14 días (6 a 12 meses) · 28 días (más de 1 año) |
| Cesantía (art. 80) | 6 días (3 a 6 meses) · 13 días (6 a 12 meses) · 21 días por año (1 a 5 años) · 23 días por año (más de 5 años) |
| Asistencia económica (art. 82) | 5 / 10 / 15 días por año (muerte o incapacidad del trabajador, cierre de la empresa, etc.) |
| Base para prestaciones | Salario promedio del último año (Reglamento 258-93) |
| Licencias | Maternidad 14 semanas (6 antes y 8 después del parto; subsidio de SISALRIL) · Paternidad 2 días (la reforma en trámite propone 4) · Matrimonio, luto y otras según el Código |
| Feriados | Ley 139-97: los que caen martes a viernes se mueven al lunes, con excepciones fijas (1 ene, 21 ene, 27 feb, 25 dic, etc.). **16 de agosto no se mueve** (Suprema Corte, 2008) |

### 1.5 Cómo termina el contrato y qué se paga en cada caso
| Causa | Preaviso | Cesantía | Asist. económica | Vacaciones y Navidad proporcionales |
|---|---|---|---|---|
| Desahucio por el empleador | ✅ | ✅ | — | ✅ |
| Despido injustificado | ✅ | ✅ | — | ✅ (+ salarios caídos si hay sentencia) |
| Despido justificado | — | — | — | ✅ |
| Dimisión justificada | ✅ | ✅ | — | ✅ |
| Renuncia (desahucio por el trabajador) | — (el trabajador debe dar preaviso) | — | — | ✅ |
| Muerte o incapacidad, cierre de la empresa | — | — | ✅ | ✅ |
| Mutuo acuerdo | Lo que se pacte | | | ✅ |

> **Reforma laboral en trámite:** en mayo de 2026 Diputados aprobó en primera lectura un proyecto que amplía la paternidad a 4 días, incluye teletrabajo y trabajo doméstico, y no cambia la cesantía. En junio volvió a comisión y en agosto de 2026 se presentó otra iniciativa en el Senado. **No es ley vigente**; el sistema debe poder adoptarla cambiando parámetros.

---

## 2. Principios de diseño
1. **Motor de cálculo puro.** Es un proyecto nuevo (`Empleados.Nomina`) sin base de datos ni web. Recibe empleado, período, incidencias y parámetros, y devuelve las líneas del recibo. Se prueba con casos que valide un contador ("golden tests").
2. **Parámetros legales versionados por fecha de vigencia:** tasas TSS, topes, escala ISR, salarios mínimos por tipo de empresa, feriados y días de vacaciones. Una nómina de marzo se calcula con los parámetros vigentes en marzo, aunque se recalcule en diciembre.
3. **Conceptos de nómina configurables.** Cada ingreso o deducción tiene sus propiedades: cotiza TSS, grava ISR, cuenta para Navidad, cuenta para prestaciones, y si es fijo, por porcentaje o por fórmula.
4. **Nóminas inmutables una vez cerradas.** Estados: Borrador → Calculada → Aprobada → Pagada → Cerrada. Los errores se corrigen con ajustes en la nómina siguiente, nunca editando una nómina ya pagada.
5. **Historial de movimientos (auditoría)** de cada cambio de salario, puesto, departamento o estado: quién, cuándo, valor anterior y valor nuevo.
6. **Dinero en `decimal`**, redondeado a 2 decimales por concepto, con una regla única y probada.
7. **Se reutiliza lo que ya existe:** servicios compartidos, roles, notificaciones, reportes, CI y fotos.

---

## 3. Fases (cada una termina en un PR con tests y CI en verde)

### Fase 3.0: Fundaciones
- Datos del empleado para nómina: cédula, NSS, AFP y ARS, tipo de contrato (indefinido, por tiempo definido, por obra), frecuencia de pago, tipo de salario (fijo mensual o por hora), banco y cuenta, dependientes del SFS, fecha de ingreso para la antigüedad.
- Empresa: RNC, razón social, tamaño (para el salario mínimo), clase de riesgo SRL y cierre fiscal. Incluye multiempresa si se decide (decisión D1).
- Parámetros legales versionados, con valores 2026 precargados y una pantalla de administración.
- Historial de movimientos.
- **Entregable:** empleados listos para calcular nómina, y una tabla de parámetros que el administrador puede actualizar sin programar.

### Fase 3.1: Motor de cálculo de la nómina regular ✅ (24-oct-2026)
- Salario del período (quincenal o mensual), con prorrateo por ingreso o salida a mitad de período.
- Descuentos de ley: AFP y SFS del empleado con sus topes, e ISR. Aportes patronales: AFP, SFS, SRL e INFOTEP.
- Horas extra (35 % y 100 %), recargo nocturno, feriados y descanso trabajado.
- Deducciones internas: préstamos y avances en cuotas, cooperativa, embargos y pensiones alimenticias con orden de prelación.
- **Entregable:** una librería probada con unos 30 casos golden, como salario en el tope del SFS, ingreso a mitad de quincena, ISR en cada tramo, o salario mínimo con horas extra.
- **Hecho:** proyecto `Empleados.Nomina`, 12 casos golden y 20 pruebas de reglas, simulador (MVC y API) y [CASOS_NOMINA.md](CASOS_NOMINA.md) con las reglas a confirmar. Las deducciones internas se reciben como líneas del período; guardarlas como préstamos en cuotas pasa a la Fase 3.3, porque las cuotas se descuentan al cerrar una nómina. Los embargos y las pensiones alimenticias quedan pendientes de definir su prelación legal.

### Fase 3.2: Incidencias y asistencia ✅ (24-oct-2026)
- Faltas, retardos, permisos con o sin goce de sueldo, licencias (médica, maternidad, paternidad, luto, matrimonio).
- Vacaciones: saldo por antigüedad, solicitud y aprobación, vacaciones colectivas.
- Calendario de feriados que aplica automáticamente las reglas de la Ley 139-97.
- Cada incidencia aprobada afecta sola a la nómina del período.
- **Hecho:** incidencias con aprobación o rechazo (Nómina → Incidencias y API), días laborables calculados sin fines de semana ni feriados, saldo de vacaciones por año de servicio (en la ficha del empleado), vacaciones colectivas por empresa o departamento, botón para generar los feriados de cualquier año y descuento automático en la nómina (los retardos en horas; las que cruzan dos quincenas se reparten). Reglas 13 a 16 de CASOS_NOMINA.md pendientes de validar.

### Fase 3.3: Ciclo de nómina ✅ (24-oct-2026)
- Crear el período, calcular, revisar diferencias contra la nómina anterior, aprobar (rol Admin o Nómina), pagar y cerrar.
- Retroactivos y ajustes.
- **Recibo de pago en PDF**, con la lista de cada ingreso y deducción.
- **Hecho:** nóminas por período con novedades por recibo, aprobación solo por Admin y bloqueada si faltan datos de TSS, reapertura, pago (que baja el saldo de los préstamos en cuotas) y cierre. Comparación del neto con la nómina anterior, PDF con QuestPDF y endpoints en la API. Los retroactivos se registran como novedades (ingreso o deducción de ajuste) en la nómina siguiente.

### Fase 3.4: Pagos especiales ✅ (24-oct-2026)
- **Salario 13 (Regalía Pascual):** cálculo proporcional y nómina especial en diciembre, sin TSS ni ISR.
- **Bonificación:** se ingresan las utilidades del año, el sistema reparte el 10 % con el tope de 45 o 60 días y aplica INFOTEP 0.5 %.
- Pago de vacaciones.
- **Hecho:** Nómina → Navidad y bonificación. El Salario de Navidad es una nómina especial (todos los empleados activos) que usa lo registrado en las nóminas del año y estima los meses sin nómina; se aprueba, se paga y se imprime como cualquier nómina. La Bonificación muestra el reparto (proporcional al salario y al tiempo trabajado, con tope de 45 o 60 días) y se carga como ingreso en una nómina regular, con la retención del INFOTEP. Las vacaciones se pagan con el salario normal de la nómina; las no disfrutadas, en la liquidación (Fase 3.5). Reglas 17 a 20 de CASOS_NOMINA.md pendientes de validar.

### Fase 3.5: Terminaciones (prestaciones laborales) ✅ (24-oct-2026)
- Asistente según la causa (tabla 1.5): preaviso, cesantía y asistencia económica según antigüedad, con el salario promedio del último año ÷ 23.83, más vacaciones y Navidad proporcionales y deducciones pendientes.
- Documentos: hoja de cálculo de prestaciones y recibo de descargo.
- Se comparan los resultados con la calculadora oficial del Ministerio de Trabajo.
- **Hecho:** Nómina → Liquidaciones (y botón en la ficha del empleado). Calcula según la causa, con el salario promedio de los últimos 12 meses (de las nóminas, o el salario actual donde no hay nómina; se puede indicar a mano), las vacaciones pendientes según el saldo más la proporción del año en curso (art. 180) y la Navidad hasta la salida (salvo que ya se haya pagado). Descuenta los saldos de préstamos. Queda en borrador; al marcarla pagada (solo Admin) el empleado sale de la empresa en la fecha de salida y los préstamos se saldan. PDF con la hoja de prestaciones y el recibo de descargo. **Pendiente:** comparar casos con la calculadora del Ministerio (reglas 21 a 25 de CASOS_NOMINA.md).

### Fase 3.6: Archivos legales y pago por banco ✅ parcial (24-oct-2026)
- TSS: archivo de autodeterminación mensual y de novedades, con el layout v24 y validación previa.
- DGII: conciliación con el IR-3.
- **Dispersión bancaria:** un formato por banco (encabezado y detalle; propio banco, ACH o LBTR). Se empieza con uno (decisión D4).
- Planilla de personal para el Ministerio de Trabajo (DGT-3 y DGT-4).
- **Hecho:** Nómina → Archivos TSS y banco. **Autodeterminación mensual (AM)** en texto de ancho fijo (E/D/S, 312 o 366 posiciones con exentos) desde las nóminas aprobadas del mes, con regalía (01) y preaviso/cesantía (02) como exentos, tipo de ingreso 0001/0004 y observaciones por trabajador. **Resumen del mes** con AFP, SFS, SRL, INFOTEP, total a pagar a la TSS e ISR retenido (cifra para conciliar el IR-3). **Pago al banco** en CSV genérico (banco, cuenta, documento, monto) y lista de quien no tiene cuenta. **Planilla de personal** en CSV como base de la DGT-3.
- **Pendiente:** (1) validar el archivo AM con el validador de estructura del SUIR: el layout se tomó de una implementación de referencia del instructivo v6.0, porque el instructivo oficial no estaba disponible en línea; (2) el formato exacto de **BHD** y Banreservas: hace falta la especificación que el banco entrega a sus clientes empresariales; (3) el archivo de novedades (NV) y la DGT-4.

### Fase 3.7: Recibos digitales, envío y portal del empleado ✅ (24-oct-2026)
- Envío automático del recibo por correo al pagar, con registro de entregado o fallido.
- Portal del empleado (decisión D6): ver recibos, saldo de vacaciones, pedir permisos y descargar la carta de trabajo.
- Notificaciones: nómina lista para aprobar, vacaciones por vencer, fin de período de prueba, cumpleaños.
- **Hecho:** al marcar una nómina pagada se envía a cada empleado su recibo en PDF por correo (SMTP de la sección `Correo`; sin servidor configurado quedan como .eml en `../correos`), con registro de enviado o fallido y botón para reenviar. **Portal del empleado** (rol Empleado; el Admin da o quita el acceso desde la ficha, con contraseña temporal por correo): recibos pagados, saldo de vacaciones, solicitudes de vacaciones y permisos (llegan por aprobar a RRHH con aviso) y carta de trabajo en PDF. Un usuario del portal no entra a ninguna otra pantalla, y la API solo da token a Admin y RRHH. **Avisos:** nómina nueva lista para aprobar, y cada día cumpleaños, fin del período de prueba (3 meses) y vacaciones que vencen en 30 días.
- **También:** página pública de inicio y acceso en español separados del sistema (sin la navegación interna).

### Fase 3.8: Reportes ✅ (24-oct-2026)
- Nómina por departamento, costo patronal total, acumulados anuales por empleado (para la declaración y para Navidad), resumen TSS e ISR del mes, ausentismo, rotación y vacaciones pendientes.
- **Hecho:** Nómina → Reportes de nómina: costo por departamento (con aportes del empleador), acumulados anuales por empleado, ausentismo, rotación mensual y vacaciones pendientes, sobre las nóminas aprobadas, con descarga en CSV (Excel) y PDF; API `/api/Reportes/nomina/{tipo}`. El resumen TSS e ISR del mes está en Archivos TSS y banco (Fase 3.6). El inicio del sistema muestra lo pendiente: nóminas y solicitudes por aprobar, liquidaciones en borrador, última nómina pagada y próximos feriados.
- Exportación a Excel y PDF.

**Orden recomendado:** 3.0 → 3.1 → 3.3 → 3.2 → 3.4 → 3.5 → 3.6 → 3.7 → 3.8. Primero calcular y pagar una nómina simple completa, de punta a punta; después enriquecerla.

---

## 4. Decisiones

### Tomadas (24-oct-2026)
- **D1:** multiempresa. Departamentos, puestos y empleados pertenecen a una empresa, y cada usuario solo ve las empresas que tiene asignadas.
- **D2:** quincenal y mensual. La empresa tiene una frecuencia por defecto y cada empleado puede tener la suya; semanal más adelante.
- **D3:** el ISR se recalcula en la 2.ª quincena con lo acumulado del mes.
- **D4:** dispersión bancaria con BHD primero y Banreservas después.
- **D5:** se sigue con Razor MVC.
- **D9:** sin contador certificado. Los casos de cálculo los valida una persona que trabaja con nóminas a diario. El sistema genera casos de prueba legibles para revisarlos.
- **D6, D7, D8 y D10:** se quedan con la recomendación (portal en la Fase 3.7, correo, QuestPDF, cambiar solo la marca en la interfaz).

### Tabla original de opciones

| # | Decisión | Opciones | Recomendación |
|---|---|---|---|
| D1 | ¿Una empresa o varias? | Una sola empresa · Multiempresa (para venderlo o para varias razones sociales) | **Multiempresa desde el modelo**: es barato ahora y caro después |
| D2 | Frecuencias de pago | Quincenal y mensual · También semanal | **Quincenal y mensual primero**; semanal después (construcción, zonas francas) |
| D3 | ISR en nómina quincenal | Mitad del ISR mensual en cada quincena · Todo en la 2.ª quincena · Recalcular en la 2.ª con lo acumulado del mes | **Recalcular en la 2.ª quincena con lo acumulado del mes** (exacto aunque el salario varíe) |
| D4 | Primer banco para la dispersión | Banreservas · Popular · BHD · Otro | El que use tu empresa o cliente; hace falta el layout que entrega el banco |
| D5 | Interfaz | Seguir con Razor MVC · Frontend SPA (React o Angular) contra la API | **Seguir con MVC** para avanzar rápido; la API ya existe por si después se quiere una SPA |
| D6 | Portal del empleado | Sí, en la Fase 3.7 · Más adelante · No | **Sí, en la 3.7** (reutiliza Identity con un rol "Empleado") |
| D7 | Envío de recibos | Correo (SMTP, SendGrid) · También WhatsApp | **Correo primero** |
| D8 | Librería de PDF | QuestPDF (gratis por debajo de US$1M de ingresos anuales) · Otra | **QuestPDF** |
| D9 | Validación legal | ¿Tienes contador o abogado laboral que valide los casos de prueba? | **Muy recomendable**; sin eso, los cálculos solo están probados contra mis lecturas de la ley |
| D10 | Nombre del producto | Mantener "Empleados" · Renombrar (por ejemplo "Nómina RD") | Renombrar solo la marca en la interfaz; los nombres de proyectos y namespaces se cambian después, si hace falta |

---

## 5. Fuentes consultadas (octubre 2026, a verificar con las oficiales)
- ISR: [Alegra, retenciones ISR 2026](https://blog.alegra.com/republica-dominicana/retenciones-isr-de-nomina/), [Alegra, cálculo ISR empleado](https://blog.alegra.com/republica-dominicana/como-calcular-el-isr-de-un-empleado/), [Infobae, recaudación ISR asalariados 2026](https://www.infobae.com/republica-dominicana/2026/03/22/el-impuesto-sobre-la-renta-de-asalariados-supera-previsiones-y-representa-el-136-de-la-recaudacion-fiscal-en-republica-dominicana/)
- TSS: [Alegra, TSS nómina 2026](https://blog.alegra.com/republica-dominicana/tss-nomina/), [G-Talent, topes 2026](https://www.g-talent.net/blogs/finanzas-administracion/tss-republica-dominicana-2026-topes-cotizacion), [Bolsillo Práctico, aportes empleador](https://bolsillopractico.com/tramites/aportes-tss-empleador-porcentajes-rd/), [TSS, instructivo de autodeterminación v24](https://tss.gob.do/assets/inst_autodeter_may_v24-5-101.pdf), [DGII, Norma 10-04](https://dgii.gov.do/legislacion/normasGenerales/Documents/NG%20sobre%20Impuesto%20sobre%20la%20Renta%20(ISR)/norma10-04.pdf)
- Salario mínimo: [Alegra, salario mínimo 2026](https://blog.alegra.com/republica-dominicana/salario-minimo-en-rd/), [EY, salario mínimo 2026](https://www.ey.com/es_ce/technical/tax/tax-alerts/republica-dominicana-salario-minimo-2026)
- Código de Trabajo: [Revista Mercado, prestaciones 2026](https://revistamercado.do/market-brief/finanzas/cuanto-te-toca-de-prestaciones-si-renuncias-o-te-despiden-en-2026-lo-que-dice-el-codigo-de-trabajo/), [Siempre al Día, calculadora de prestaciones](https://siemprealdia.co/republica-dominicana/herramientas/calculadora-de-prestaciones-laborales-en-republica-dominicana/), [Phlaw, bonificación](https://phlaw.com/es/participacion-en-utilidades-empresa-bonificacion/), [Phlaw, aspectos del Código](https://phlaw.com/es/34principales-aspectos-codigo-trabajo-en-republica-dominicana34/), [Decreto 258-93](https://transparencia.mt.gob.do/images/docs/base_legal/decretos/decreto-258-93-aplicacion-ct.pdf), [Hacecuentas, recargo nocturno](https://hacecuentas.com/do/calculadora-recargo-nocturno-republica-dominicana), [El Caribe, pago en feriados](https://www.elcaribe.com.do/panorama/dinero/cuanto-pagan-si-trabajo-en-semana-santa-2024-en-republica-dominicana/)
- Licencias: [El Caribe, maternidad 14 semanas](https://www.elcaribe.com.do/panorama/pais/subsidio-por-maternidad-sera-por-14-semanas/), [Reglamento de maternidad y lactancia](https://dida.panel-dev-pys.ogtic.gob.do/wp-content/uploads/2024/05/Reglamento-sobre-el-Subsidio-por-Maternidad-y-el-Subsidio-por-Lactancia-Modificado-Res.-Congreso-Nacional-211-.docx)
- Feriados: [El Caribe, feriados 2026](https://www.elcaribe.com.do/panorama/pais/dias-feriados-republica-dominicana-2026/), [Revista Mercado, 16 de agosto](https://revistamercado.do/money-invest/daily-news/habra-fin-de-semana-largo-el-ministerio-de-trabajo-aclara-si-el-feriado-del-16-de-agosto-se-corre-al-lunes-17/)
- Reforma laboral: [Diario Libre, mayo 2026](https://www.diariolibre.com/politica/congreso-nacional/2026/05/20/aprueban-reforma-laboral-con-mas-licencia-de-paternidad/3540367), [Siempre al Día, reforma 2026](https://siemprealdia.co/republica-dominicana/derecho-laboral/reforma-al-codigo-de-trabajo-en-republica-dominicana/)
- Banca: [Banesco, plantilla de pago de nómina](https://www.banesco.com.do/wp-content/uploads/2023/10/Guia-Plantilla-Pago-de-Nomina-BanescOnline.pdf)
