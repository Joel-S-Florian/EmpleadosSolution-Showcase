# Casos de cálculo de nómina para validar

Este documento es para que **quien hace nóminas a diario** revise si el sistema calcula como se hace en la práctica.
Cada caso tiene sus datos y el resultado del sistema. Los mismos casos están como pruebas automáticas (`Empleados.Tests/NominaGoldenTests.cs`): si algún resultado no coincide con lo que tú calcularías, se corrige la regla y la prueba.

También puedes probar casos propios en la pantalla **Nómina → Simulador**. Ahí se muestra paso a paso cómo se llegó a cada monto.

## Parámetros usados (2026)
| Concepto | Empleado | Empleador | Tope mensual |
|---|---|---|---|
| AFP | 2.87 % | 7.10 % | RD$464,460.00 (20 × 23,223) |
| SFS | 3.04 % | 7.09 % | RD$232,230.00 (10 × 23,223) |
| Riesgos laborales (SRL) | — | 1.20 % (según la empresa) | RD$92,892.00 (4 × 23,223) |
| INFOTEP | — | 1 % | sin tope |

**ISR (escala anual):** exento hasta 416,220.00; 15 % hasta 624,329.00; 31,216.00 + 20 % hasta 867,123.00; 79,776.00 + 25 % por encima.

**Salario diario** = salario mensual ÷ 23.83. **Valor hora** = salario diario ÷ 8.

## Reglas que conviene confirmar ✋
Marca cada una con ✅ si es como lo haces, o anota cómo lo haces tú:

1. **Quincenas:** la 1.ª va del 1 al 15 y la 2.ª del 16 al fin de mes. Cada quincena paga la mitad del salario mensual.
2. **Ingreso o salida a mitad de período:** el salario se paga proporcional a los **días calendario** trabajados (ejemplo: entra el 24, en una 2.ª quincena de 16 días cobra 8/16 de la quincena). ¿Lo haces por días calendario, por días laborables, o con el salario diario (÷ 23.83)?
3. **Ausencias sin pago:** se descuentan con el salario diario legal (mensual ÷ 23.83 × días).
4. **TSS en quincenas:** en la 1.ª quincena se retiene sobre lo de esa quincena. En la 2.ª se calcula el aporte del **mes completo** (con el tope mensual) y se resta lo ya retenido.
5. **ISR en quincenas:** en la 1.ª quincena se proyecta el mes (lo de la quincena × 2) y se retiene la mitad. En la 2.ª se calcula el ISR del mes completo y se retiene la diferencia. Si la 1.ª retuvo de más (por ejemplo, por un bono), la 2.ª **devuelve** la diferencia (caso L).
6. **Horas extra:** valor hora × 1.35 (hasta 68 h semanales) o × 2 (por encima de 68 h).
7. **Horas nocturnas y feriados:** solo se paga el **recargo** (15 % o 100 %), porque esas horas ya están pagadas dentro del salario. ¿Es así, o pagas el feriado trabajado completo aparte?
8. **INFOTEP 1 %:** sobre todo lo que cotiza a la TSS en el período (salario, extras, bonos).
9. **Dependientes adicionales del SFS:** se descuentan en el último pago del mes. **Falta configurar su costo** en Parámetros legales; ¿cuánto es hoy por dependiente?
10. **Deducciones internas** (préstamos, cooperativa): se aplican después de las de ley, en orden de prioridad, sin dejar el neto en negativo. Lo que no alcanza queda como aviso.
11. **Redondeo:** cada monto se redondea a 2 decimales (0.005 sube).
12. **Límite entre los tramos del 20 % y el 25 % del ISR:** la DGII publica 79,776 de cuota fija, pero el acumulado exacto da 79,774.80. El sistema usa la tabla publicada, igual que la DGII.
13. **Días laborables de una incidencia:** se cuentan de lunes a viernes, sin feriados. Si la empresa trabaja sábados, los días se ajustan a mano. ¿Se cuenta el sábado como medio día?
14. **Qué se descuenta por defecto:** faltas, retardos (horas ÷ 8 = fracción de día) y permisos sin goce. Las licencias (médica, maternidad, paternidad, luto, matrimonio) se pagan, salvo que se marque "descontar". ¿Se descuenta la licencia médica cuando la cubre el subsidio de la SISALRIL?
15. **Saldo de vacaciones:** 14 días al cumplir de 1 a 4 años y 18 desde los 5 años (art. 177). El saldo es el del año de servicio en curso; lo que no se tomó el año anterior no pasa al siguiente. ¿Se permite arrastrar días?
16. **Incidencia aprobada después de aprobar la nómina:** no cambia esa nómina; el sistema avisa para registrar el descuento como novedad en la siguiente.
17. **Salario de Navidad:** salario ordinario = salario menos ausencias; no incluye horas extra, comisiones ni bonificaciones. Los meses sin nómina en el sistema se estiman con el salario actual. ¿Se incluyen las comisiones?
18. **Bonificación, reparto:** el 10 % de la utilidad se reparte en proporción al salario mensual y al tiempo trabajado en el ejercicio. Lo que excede el tope de un empleado no se redistribuye. ¿Es así como lo hacen?
19. **Bonificación, tope:** 45 días de salario (menos de 3 años al cierre) o 60 días (3 años o más), proporcional al tiempo trabajado en el ejercicio. Los despedidos antes del pago no participan aquí.
20. **Bonificación, impuestos:** grava ISR (se suma a los ingresos del mes en que se paga), no cotiza a la TSS y se retiene 0.5 % para el INFOTEP.
21. **Cesantía con fracción de año:** de 1 a 5 años, 21 días por año; desde 5 años, 23 días por **todos** los años. La fracción de más de 3 meses suma 6 días (3 a 6 meses) o 13 días (6 meses o más). ¿Coincide con la calculadora del Ministerio?
22. **Salario promedio:** promedio del salario ordinario de los 12 meses completos antes del mes de salida (sin horas extra ni bonos). ¿Se incluyen las comisiones y las horas extra habituales?
23. **Vacaciones al salir:** las pendientes del año de servicio ya cumplido más la proporción del año en curso según el art. 180 (6 días con 5 meses, hasta 12 con 11 meses), también para quien tiene más de 5 años. Todo se paga con el salario diario promedio.
24. **Impuestos de la liquidación:** preaviso, cesantía, asistencia y Navidad no pagan ISR ni TSS. El sistema tampoco retiene sobre las vacaciones. ¿Se retiene algo sobre las vacaciones pagadas al salir?
25. **Préstamos:** el saldo se descuenta de la liquidación hasta donde alcance; lo que no alcanza queda como aviso.

## Casos
Todos son del mes de **octubre 2026** y de empresas con SRL de 1.20 %.

### A. Salario mensual de 35,000 (exento de ISR)
| Concepto | Monto |
|---|---|
| AFP (35,000 × 2.87 %) | 1,004.50 |
| SFS (35,000 × 3.04 %) | 1,064.00 |
| Base ISR: 35,000 − 1,004.50 − 1,064.00 = 32,931.50 × 12 = 395,178.00 al año | exento |
| **Neto** | **32,931.50** |
| Empleador: AFP 2,485.00 · SFS 2,481.50 · SRL 420.00 · INFOTEP 350.00 | |
| **Costo total para la empresa** | **40,736.50** |

### B. Salario mensual de 60,000 (tramo del 20 %)
| Concepto | Monto |
|---|---|
| AFP | 1,722.00 |
| SFS | 1,824.00 |
| Base ISR: 60,000 − 1,722 − 1,824 = 56,454.00 × 12 = 677,448.00 | |
| ISR anual: 31,216 + (677,448 − 624,329) × 20 % = 41,839.80 ÷ 12 | 3,486.65 |
| **Neto** | **52,967.35** |
| **Costo total para la empresa** | **69,834.00** |

### C. El mismo salario de 60,000, pagado por quincena
| | 1.ª quincena | 2.ª quincena |
|---|---|---|
| Salario | 30,000.00 | 30,000.00 |
| AFP | 861.00 | 861.00 |
| SFS | 912.00 | 912.00 |
| ISR | 1,743.33 (mitad del proyectado: 3,486.65 ÷ 2) | 1,743.32 (3,486.65 − 1,743.33) |
| **Neto** | **26,483.67** | **26,483.68** |

Entre las dos quincenas suman 52,967.35, lo mismo que el caso B.

### D. Salario mensual de 300,000 (supera los topes del SFS y del SRL)
| Concepto | Monto |
|---|---|
| AFP (300,000 × 2.87 %, no llega al tope) | 8,610.00 |
| SFS (tope 232,230 × 3.04 %) | 7,059.79 |
| Base ISR: 300,000 − 8,610.00 − 7,059.79 = 284,330.21 × 12 = 3,411,962.52 | |
| ISR anual: 79,776 + (3,411,962.52 − 867,123) × 25 % = 715,985.88 ÷ 12 | 59,665.49 |
| **Neto** | **224,664.72** |
| Empleador: AFP 21,300.00 · SFS (tope) 16,465.11 · SRL (tope 92,892) 1,114.70 · INFOTEP 3,000.00 | |
| **Costo total para la empresa** | **341,879.81** |

### E. 400,000 mensual por quincenas (el tope del SFS se ajusta en la 2.ª)
| | 1.ª quincena | 2.ª quincena |
|---|---|---|
| Salario | 200,000.00 | 200,000.00 |
| AFP | 5,740.00 | 5,740.00 |
| SFS | 6,080.00 (200,000 × 3.04 %) | 979.79 (tope del mes 7,059.79 − 6,080.00) |
| ISR | 41,336.47 | 42,611.52 |
| **Neto** | **146,843.53** | **150,668.69** |

### F. Entra a trabajar el 24 de octubre (2.ª quincena, 30,000 mensual)
- Quincena: 15,000. Trabajó 8 de los 16 días → **7,500.00**
- AFP 215.25 · SFS 228.00 · ISR exento → **Neto 7,056.75**

### G. Salario mínimo (29,988) con horas extra, nocturnas y feriado
Valor hora: 29,988 ÷ 23.83 ÷ 8 = 157.3017
| Concepto | Cálculo | Monto |
|---|---|---|
| 10 horas extra (35 %) | 157.3017 × 10 × 1.35 | 2,123.57 |
| 2 horas extra sobre 68 h (100 %) | 157.3017 × 2 × 2 | 629.21 |
| 8 horas nocturnas (recargo 15 %) | 157.3017 × 8 × 0.15 | 188.76 |
| 8 horas en feriado (recargo 100 %) | 157.3017 × 8 × 1 | 1,258.41 |
| Total ingresos | | 34,187.95 |
| AFP · SFS | | 981.19 · 1,039.31 |
| **Neto** (exento de ISR) | | **32,167.45** |

### H. 35,000 mensual con 2 días de ausencia sin pago
- Ausencia: 35,000 ÷ 23.83 × 2 = **−2,937.47**
- AFP 920.19 · SFS 974.70 · ISR exento → **Neto 30,167.64**

### I. Salario por hora: 180 la hora, 88 horas en la 1.ª quincena
- Salario 180 × 88 = **15,840.00** · AFP 454.61 · SFS 481.54 → **Neto 14,903.85**

### J. Deducciones internas que no alcanzan (35,000 mensual)
- Disponible después de las deducciones de ley: 32,931.50
- Préstamo (prioridad 1): 30,000.00, completo
- Cooperativa (prioridad 2): 5,000.00 → se descuentan solo **2,931.50** y quedan **2,068.50 sin descontar** (aviso)
- **Neto: 0.00**

### K. Bono de 50,000 en la 1.ª quincena (80,000 mensual)
| | 1.ª quincena (40,000 + bono 50,000) | 2.ª quincena (40,000) |
|---|---|---|
| AFP | 2,583.00 | 1,148.00 |
| SFS | 2,736.00 | 1,216.00 |
| ISR | 15,461.72 | 3,700.47 |
| **Neto** | **69,219.28** | **33,935.53** |

### L. Devolución de ISR en la 2.ª quincena (40,000 mensual + bono 50,000 en la 1.ª)
| | 1.ª quincena (20,000 + bono 50,000) | 2.ª quincena (20,000) |
|---|---|---|
| AFP | 2,009.00 | 574.00 |
| SFS | 2,128.00 | 608.00 |
| ISR | 10,757.22 (proyecta como si cobrara 70,000 dos veces) | **−1,004.03 (devolución)**: el ISR real del mes es 9,753.19 |
| **Neto** | **55,105.78** | **19,822.03** |

¿En la práctica devuelves en la 2.ª quincena lo retenido de más, o lo compensas el mes siguiente?
