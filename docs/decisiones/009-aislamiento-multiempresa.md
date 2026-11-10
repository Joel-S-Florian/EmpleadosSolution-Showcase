# ADR 009: Aislamiento entre empresas (SaaS)

**Contexto.** Se decidió que el producto sea un SaaS multiempresa ([ADR 008](008-decisiones-de-producto.md)). Hasta ahora "multiempresa" era para un despacho que lleva varias empresas: el filtro global de EF aislaba los datos, pero los roles eran globales y las pantallas de empresas asumían un solo administrador de todo. Antes de añadir registro y planes se auditó qué se podía ver o tocar entre empresas.

**Qué se auditó y qué se encontró.**

| Hallazgo | Riesgo | Qué se hizo |
|---|---|---|
| Todas las tablas con `EmpresaId` tienen filtro global | Ninguno | Una prueba lo exige para cualquier tabla nueva |
| 8 usos de `IgnoreQueryFilters` | Cada uno es una vía a datos de todas las empresas | Una prueba fija la lista y el número por archivo; uno nuevo la rompe |
| `EmpresaService` listaba, leía y editaba **cualquier** empresa | Un Admin de la empresa A veía y editaba la B | Todo acotado a las empresas del usuario (`Alcance()`) |
| La pantalla de accesos listaba **todos los usuarios** del sistema | Se filtraban correos de otras empresas y se podían asignar | Solo se ven los usuarios de mis empresas; se rechaza asignar otros |
| Cualquier Admin podía crear empresas | En SaaS las crea la plataforma o el registro | Solo `SuperAdmin` |
| Cualquier Admin podía cambiar parámetros legales (TSS, ISR, feriados) | Cambiaría la ley para todas las empresas | Solo `SuperAdmin`; el resto ve la pantalla en solo lectura |
| Fotos en `/uploads` sin sesión, con nombre `id_fechaHora` | Se podía adivinar la foto de otro empleado | El nombre lleva un GUID |
| Notificaciones con destinatario nulo las ve cualquiera | Mensaje global entre empresas | Solo las crea la semilla de desarrollo; sin cambio |

**Modelo de roles.** `SuperAdmin` (la plataforma) ve y gestiona todas las empresas y la ley; `Admin` administra las empresas a las que pertenece; `RRHH` opera dentro de ellas; `Empleado` usa el portal. La política `Plataforma` exige `SuperAdmin`. El administrador de prueba (solo con contraseña configurada en desarrollo) recibe `SuperAdmin`.

**Lo que NO se hizo (y cuándo hacerlo).**
- **Roles por empresa.** Los roles siguen siendo globales: quien es Admin en la empresa A y está invitado a la B como RRHH es Admin en ambas. Hoy cada usuario pertenece a una sola empresa; si un usuario debe estar en varias con roles distintos, el rol pasa a `UsuarioEmpresa`.
- **Los datos de `/uploads` siguen sin sesión.** El nombre no adivinable reduce el riesgo, pero no es control de acceso; servirlas por un controlador autorizado es el cambio completo.
- **Primer SuperAdmin en producción.** No hay forma de crearlo desde la aplicación a propósito; se asigna por base de datos o por la semilla de desarrollo.

**Alternativas descartadas.** Una base de datos o un esquema por empresa: más aislamiento, pero migraciones y costo operativo desproporcionados para este tamaño. Seguir confiando solo en el filtro global: no cubre las tablas sin `EmpresaId` ni los accesos que lo saltan, justo donde estaban las fugas.

**En entrevista (30 s).** "Antes de abrirlo a más clientes audité el aislamiento: el filtro por empresa cubría los datos, pero la administración de empresas y de usuarios era global. Lo cerré con un alcance por usuario y un rol de plataforma, y dejé pruebas que fallan si alguien añade una tabla sin filtro o un `IgnoreQueryFilters` nuevo."

## Addendum: registro, usuarios y planes (7.2 y 7.3)

- **Registro de empresa** (`/Registro`): crea el usuario, la empresa en **plan de prueba** (10 empleados) y le da el rol `Admin`; limitado a 5 intentos por hora y por IP. Está **cerrado por defecto** (`Plataforma:RegistroAbierto`), abierto solo en desarrollo. Quien se registra no puede elegir su plan.
- **Usuarios de la empresa:** el administrador crea usuarios `Admin` o `RRHH` directamente en su empresa (ficha de la empresa), en vez de elegir entre todos los usuarios del sistema.
- **Planes** (`Empresa.Plan`): Prueba 10, Básico 50, Profesional 250 empleados; solo la plataforma lo cambia. Crear un empleado por encima del límite se rechaza con un mensaje claro; los despedidos no cuentan. Las empresas que ya existían quedaron en Profesional.

**Riesgos conocidos al abrir el registro al público.**
- **Sin confirmación de correo:** cualquiera puede registrar una empresa con un correo ajeno. Antes de abrirlo hay que confirmar el correo (necesita un servidor de correo real).
- **Sin recuperación de contraseña** ni cambio forzado de la contraseña inicial de los usuarios creados por el administrador.
- **Los límites del plan** solo se aplican al crear empleados; no bloquean una empresa que baje de plan con más empleados de los permitidos, ni hay cobros.
- El límite por IP detrás de un proxy usa la IP del proxy (`shortcut:` en el código).
