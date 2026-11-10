# Plan de mejoras: arquitectura, calidad y explicabilidad

> Estado: **borrador para decidir**. Cada fase deja la solución compilando y con las 170 pruebas en verde, y termina con una decisión que se puede defender en una entrevista (ver "Cómo se explica").

## 0. Punto de partida (auditado el 2026-11-09)

**Lo que ya está bien y NO se toca** (es lo que se vende):

| Fortaleza | Evidencia |
|---|---|
| Motor de nómina como función pura, sin BD ni reloj | `Empleados.Nomina/MotorNomina.cs`: `Calcular(entrada, parametros) -> recibo` |
| Parámetros legales versionados por fecha, no hardcodeados | `ParametrosLegales`, `ParametrosLegalesService` |
| Resultado explícito en vez de excepciones para reglas de negocio | `Resultado` / `Resultado<T>`, `ResultadoExtensions` |
| Pruebas en capas: unitarias del motor, *golden tests*, integración con EF InMemory | 170 pruebas, 15 s, todas pasan |
| CI que además verifica migraciones pendientes desde los dos hosts | `.github/workflows/ci.yml` |
| Controladores delgados (API y MVC solo orquestan) | `EmpleadosController` |

**Debilidades reales encontradas** (cada una es una fase del plan):

| # | Hallazgo | Evidencia | Riesgo |
|---|---|---|---|
| D1 | `Empleados.Data` hace de todo: entidades, DbContext, servicios de negocio, PDF, Excel, correo | 25+ servicios y generadores de PDF en el mismo proyecto que las migraciones | Nombre engañoso; no se puede explicar "capas" con honestidad |
| D2 | Solo 2 interfaces (`IContextoUsuario`, `IEnviadorCorreo`); servicios inyectados como clases concretas | `ServiceCollectionExtensions.cs` | Los controladores y servicios dependen de implementaciones; las pruebas no pueden aislar |
| D3 | Reloj del sistema usado directo en 12+ lugares | `DateTime.Now/Today/UtcNow` en controladores, `AuthService`, PDF, `Notificacion` | Pruebas no deterministas; fechas de nómina dependen del servidor |
| D4 | Entidades anémicas: las reglas de estado viven en el servicio, no en `Empleado` | `EmpleadoService.ActualizarAsync` revisa `Estado == Despedido` | Regla duplicable y fácil de saltarse |
| D5 | Lecturas sin `AsNoTracking` y con `Include` + entidad completa; el STAR dice "proyección `.Select()`", pero `ListarAsync` ya no lo hace | `EmpleadoService.ListarAsync`, 0 usos de `AsNoTracking` | Rendimiento; el CV/STAR afirma algo que el código ya no cumple |
| D6 | Filtro `e.FechaContratacion.Date == x` | `ListarAsync` | No usa índice (no sargable) |
| D7 | Casi sin logging estructurado (4 usos de `ILogger`) | grep | Imposible diagnosticar un cálculo de nómina en producción |
| D8 | Dos mecanismos de auth distintos (Identity con cookie en MVC, JWT en API) sin documentar el porqué | `Empleados.Mvc/Program.cs`, `Empleados.Api/Program.cs` | Pregunta segura de entrevista |
| D9 | README, CV y STAR describen la versión vieja | README: "tres capas", sin nómina | Desalineado con lo que se muestra |
| D10 | Pruebas de integración con `UseInMemoryDatabase`, que no valida SQL real (transacciones, collation, restricciones) | `Empleados.Tests` | Falsa confianza en consultas |

## 1. Principios (restricciones del plan)

1. **Ningún cambio de comportamiento** salvo corregir un bug probado. Las 170 pruebas actuales son la red de seguridad; no se editan para que pasen.
2. **Un refactor por PR**, rama `mejoras/<tema>`, CI en verde antes de seguir.
3. **Solo lo que se pueda explicar en 2 minutos y se vea en el código.** Sin MediatR, sin CQRS, sin microservicios: cada capa de más debe justificar su costo.
4. Cada fase deja un **ADR** corto en `docs/decisiones/` (contexto, decisión, alternativas descartadas, consecuencias). Eso es el guion de entrevista.

## 2. Fases

### Fase 1: Reloj inyectable (D3) · esfuerzo S
- Reemplazar `DateTime.Now/Today/UtcNow` por `TimeProvider` (ya viene en .NET 8+, sin paquete nuevo).
- Pruebas con `FakeTimeProvider` (`Microsoft.Extensions.TimeProvider.Testing`) para fechas de nómina, vencimiento de préstamos y recordatorios.
- **Por qué primero:** es mecánico, bajo riesgo y elimina la causa de pruebas frágiles.
- **Cómo se explica:** "El motor ya era puro; extendí la misma idea al resto: nada lee el reloj directamente."

### Fase 2: Lecturas eficientes (D5, D6) · esfuerzo S
- Listados y reportes: `AsNoTracking()` y proyección `.Select()` a DTO donde hoy se devuelve la entidad completa.
- Filtro por fecha como rango (`>= dia && < dia+1`) para poder usar índice.
- Prueba que fije el contrato de paginación; revisar el SQL generado con `ToQueryString()` en una prueba.
- **Cómo se explica:** "Medí/leí el SQL generado, no asumí. Esto además hace verdadero lo que dice el STAR."

### Fase 3: Dominio con comportamiento (D4) · esfuerzo M · HECHA (ver ADR 003)
- Mover transiciones de estado a `Empleado`: `Desactivar()`, `Reactivar()`, `Despedir(fecha)`, devolviendo `Resultado`. Setters de `Estado` y `FechaTerminacion` pasan a `private`/`init` donde EF lo permita.
- `EmpleadoService` orquesta (carga, llama al método, guarda, notifica); ya no contiene la regla.
- **Alcance deliberado:** solo `Empleado` (y `NominaPeriodo` si hay transiciones de estado tipo Borrador→Calculada→Cerrada). No se rediseña todo a DDD.
- **Cómo se explica:** "Regla de negocio dentro de la entidad: no hay forma de saltársela desde otro servicio. Lo apliqué donde había invariantes reales, no por moda."

### Fase 4: Separar capas de verdad (D1, D2) · esfuerzo L · HECHA EN VERSIÓN REDUCIDA (ver ADR 004: se separó Documentos; Dominio/Aplicación/Infraestructura se difiere con su condición de revisión)
Estructura objetivo (dependencias hacia adentro):

```
Empleados.Nomina        (motor puro, sin dependencias)               ← ya existe
Empleados.Dominio       (entidades, enums, Resultado, interfaces)    ← nuevo, depende de Nomina si hace falta
Empleados.Aplicacion    (servicios de negocio, DTOs)                 ← nuevo, depende de Dominio
Empleados.Infraestructura (EF Core, migraciones, PDF, Excel, correo) ← renombrar Empleados.Data
Empleados.Api / Empleados.Mvc (hosts)
```

Pasos (cada uno un PR, sin mover todo de golpe):
1. Extraer `Resultado`, `Paginado`, interfaces y enums a `Empleados.Dominio`.
2. Interfaces solo donde hay **frontera externa o razón de prueba**: `IGeneradorPdf`, `IExportadorExcel`, `IEnviadorCorreo` (ya existe), `IAlmacenFotos`, `IReloj` (Fase 1). **No** `IEmpleadoService` por cada servicio: sería ceremonia.
3. Mover PDF/Excel/correo/fotos a Infraestructura; los servicios de aplicación dependen de las interfaces.
4. Opcional y discutible: **no** introducir repositorios sobre EF. `DbContext` ya es unit of work + repositorio; envolverlo agrega código sin aislar nada. Documentarlo en un ADR (es una decisión que los entrevistadores suelen cuestionar).

- **Cómo se explica:** "Hoy `Data` mezclaba responsabilidades. Separé por dirección de dependencias, y abstraje solo lo que cambia (PDF, correo, fotos) o lo que estorba al probar (reloj)."

### Fase 5: Pruebas contra SQL Server real (D10) · esfuerzo M · HECHA (ver ADR 005; variable de entorno en vez de Testcontainers; encontró y corrigió un bug de concurrencia)
- Suite de integración con **Testcontainers (SQL Server)** para: migraciones desde cero, paginación/filtros, restricciones de integridad, concurrencia en el cierre de nómina.
- Dejar EF InMemory solo para pruebas rápidas de lógica; marcar las de contenedor con `[Trait("Categoria","Integracion")]` y correrlas en CI en un job aparte.
- **Cómo se explica:** "InMemory no es relacional: no valida restricciones ni SQL. Usé contenedor real para lo que depende del motor."

### Fase 6: Observabilidad (D7) · esfuerzo S · HECHA (ver ADR 006)
- `ILogger` con plantillas estructuradas en el ciclo de nómina (inicio/fin de cálculo, empleado, período, advertencias del motor) y en liquidaciones. **Sin datos personales ni salarios en los logs.**
- Manejo global de errores (`IExceptionHandler` + `ProblemDetails`) en la API.
- Endpoint `/health` con chequeo de BD.
- **Cómo se explica:** "Para depurar una nómina en producción necesito poder reconstruir qué se calculó y con qué parámetros, sin exponer salarios."

### Fase 7: Seguridad y autenticación explicadas (D8) · esfuerzo S · HECHA (ver ADR 007; refresh tokens y CSP quedan pendientes con su condición)
- ADR: por qué MVC usa Identity con cookie (aplicación server-rendered, el navegador gestiona la sesión) y la API usa JWT (clientes externos sin estado).
- Revisar: expiración y *refresh* del JWT, política de contraseñas, límite de intentos (`rate limiting` de .NET), cabeceras de seguridad, autorización por política en vez de `Roles = "Admin"` repetido.
- **Cómo se explica:** "Dos hosts, dos mecanismos, cada uno por una razón concreta."

### Fase 8: Alineación de narrativa (D9) · esfuerzo S · HECHA (README nuevo con arquitectura, decisiones y capturas; bullet del CV y STAR del Proyecto 3 actualizados fuera del repo)
- README nuevo: qué es, diagrama de capas (Mermaid), cómo correr, decisiones clave enlazadas a los ADR, capturas.
- Actualizar bullet del CV y reescribir el STAR del Proyecto 3 con lo construido (motor puro, parámetros versionados, golden tests, CI, separación de capas).

## 3. Orden y dependencias

`Fase 1 → Fase 2` (independientes, rápidas, empezar por aquí) → `Fase 3` → `Fase 4` (usa el reloj de la Fase 1 como primera interfaz) → `Fase 5` → `Fase 6` y `Fase 7` (en paralelo) → `Fase 8` (al final, cuando el código ya dice la verdad).

## 4. Fuera de alcance (a propósito)

- Pasar a microservicios, CQRS, MediatR, Clean Architecture "de libro" con casos de uso por clase: costo alto, sin problema real que lo justifique en este tamaño (~13k líneas).
- Frontend SPA: el MVC cumple y la API ya existe.
- Cambiar de SQL Server: no aporta.

## 5. Criterio de "terminado" por fase

1. `dotnet build` y `dotnet test` en verde, sin editar pruebas existentes (salvo las de la propia fase).
2. `dotnet ef migrations has-pending-model-changes` limpio desde Api y Mvc.
3. Un ADR escrito con alternativas descartadas.
4. Una frase de entrevista de 20 segundos y una de 2 minutos en el ADR.

## 6. Preguntas abiertas para decidir

1. ¿Aceptas separar en 5 proyectos (Fase 4) o prefieres 3 (Nomina, Data+servicios, hosts) y documentar la limitación? Más proyectos = mejor historia de capas, más navegación.
2. ¿El MVC debe seguir accediendo a servicios directamente (como ahora) o consumir la API por HTTP? Lo primero es más simple; lo segundo es más "arquitectura de dos apps independientes" pero duplica trabajo.
3. ¿Hay despliegue previsto? Si sí, conviene adelantar Fase 6 (logs/health) y un `Dockerfile`.

## 7. Cola (por planificar, añadido 2026-11-09)

Estos tres puntos no están diseñados todavía: cada uno empieza con una **fase de descubrimiento corta** (inventario y decisión) antes de tocar código.

### Fase 9: Paginación en los módulos que tienden a crecer · esfuerzo M
- **Descubrimiento:** inventariar qué listados devuelven todo (`ToListAsync()` sin `Take`) y cuáles crecen sin límite con el uso: nóminas y sus recibos, liquidaciones, incidencias, préstamos, notificaciones, movimientos de empleados, correos enviados. Los catálogos pequeños (departamentos, puestos, bancos) no se tocan.
- **Decisión a tomar:** reutilizar `Paginado<T>` y el patrón de `EmpleadoService` (Fase 2: `NormalizarPagina` + proyección) o extraer un helper común. Paginación por `Skip/Take` (simple, suficiente aquí) frente a paginación por cursor (solo si hay listados con mucho volumen y orden estable).
- **Cómo se explica:** "Paginé donde el volumen crece con el tiempo, con tope máximo de página en el servidor; los catálogos acotados no."

### Fase 10: Rediseño de notificaciones · esfuerzo M, requiere planificación
- **Descubrimiento:** qué se notifica hoy (`NotificacionService`, `RecordatoriosService`, `RecordatoriosDiarios`, panel del MVC), a quién y cuándo; qué ruido genera; qué falta (leídas/no leídas, preferencias, correo, agrupación, expiración).
- **Preguntas a decidir antes de diseñar:** ¿en proceso (hoy) o con cola y reintentos?; ¿canal único (campana) o varios (correo)?; ¿cómo evitar duplicados de recordatorios (hoy se comprueba por mensaje y fecha)?; ¿retención/limpieza?
- Debe apoyarse en la Fase 6 (logging) y en `TimeProvider` (Fase 1) para pruebas de recordatorios.

### Fase 11: Simplificar y documentar los flujos de trabajo · esfuerzo M
- **Descubrimiento (primero entender, luego simplificar):** documentar con diagramas Mermaid los flujos reales: ciclo de nómina (Borrador → Calculada → Aprobada → Pagada → Cerrada, 2.ª quincena depende de la 1.ª), incidencias/vacaciones, liquidaciones, préstamos, portal del empleado. Dónde está cada regla y qué servicio la ejecuta.
- **Candidatos a revisar por complejidad:** `CicloNominaService` (~530 líneas), `GeneradorReportes` (~450), `IncidenciaService`, `LiquidacionService`; flujos con muchos estados o pasos manuales.
- **Salida esperada:** un documento `docs/FLUJOS.md` con un diagrama por flujo, una lista de simplificaciones propuestas ordenadas por beneficio/riesgo (p. ej. máquinas de estado explícitas como en la Fase 3), y solo después cambios de código, uno por PR.
- **Cómo se explica:** "Antes de simplificar, dibujé los flujos para ver qué complejidad es del negocio (leyes laborales) y cuál es accidental."

> Revisión de 2026-11-09: las dudas de usabilidad, lo ya cambiado y las decisiones pendientes están en [FLUJOS.md](FLUJOS.md).

### Fase 12: SaaS multiempresa (decisión 1 del [ADR 008](decisiones/008-decisiones-de-producto.md)) · esfuerzo L, requiere planificación
- **Ya existe:** filtro global de EF por empresa y `UsuarioEmpresa`, con pruebas de aislamiento (`MultiempresaTests`).
- **Falta decidir y construir:** alta de empresas por autoservicio, quién es el dueño y quién administra, planes y límites (empleados por plan), aislamiento más allá del filtro (pruebas por endpoint, el archivo de fotos y los respaldos por empresa), y qué ve un superadministrador.
- **Descubrimiento primero:** inventariar cada consulta que ignora el filtro (`IgnoreQueryFilters`), cada recurso fuera de la base de datos (fotos, archivos) y el modelo de roles.

### Fase 13: Respaldos recurrentes y descarga (decisión 6) · esfuerzo M
- Respaldo programado en el servidor con retención de N días; descarga solo para Admin, con confirmación y registro en el log.
- **Riesgos a diseñar:** `BACKUP DATABASE` necesita permisos en SQL Server y una carpeta accesible por el motor; la descarga expone todos los datos (con SaaS, de todas las empresas: solo un superadministrador, o respaldo por empresa en formato exportable).
- **Dependencia:** conviene hacerla después de la Fase 12 para saber si el respaldo es por empresa o global.

> Las fases 9 a 13 se desarrollan, con alcance y orden, en [PLAN_PRODUCTO.md](PLAN_PRODUCTO.md).
