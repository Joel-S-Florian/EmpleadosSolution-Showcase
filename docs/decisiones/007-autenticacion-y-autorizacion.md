# ADR 007: Autenticación y autorización

**Contexto.** Hay dos aplicaciones sobre la misma base de usuarios (ASP.NET Core Identity) pero con mecanismos distintos, y no estaba escrito por qué. Además: la API no bloqueaba cuentas por intentos fallidos, tenía otra política de contraseñas que el MVC, las cadenas de roles (`Roles = "Admin"`, `"Admin,RRHH"`) estaban repetidas en 55 atributos y el MVC definía dos políticas (`AdminOnly`, `RRHHOnly`) que nadie usaba.

**Por qué dos mecanismos.**
- **MVC → Identity con cookie.** Es una aplicación renderizada en el servidor: el navegador gestiona la sesión, la cookie de Identity es `HttpOnly` (el JavaScript de la página no puede leerla) y el inicio de sesión ya trae bloqueo de cuenta.
- **API → JWT Bearer.** Es para clientes que no son un navegador de esta aplicación (integraciones, otro frontend): sin estado, el cliente envía el token en cada llamada y el servidor solo valida la firma. Solo Admin y RRHH reciben token; el rol Empleado (portal) no.

**Decisión.**
- **Políticas con nombre**, definidas una vez (`Politicas.SoloAdmin`, `Politicas.RRHH` en `Empleados.Data/Services/Seguridad.cs`) y usadas por los dos hosts: `[Authorize(Policy = Politicas.SoloAdmin)]`. Cambiar quién puede qué es tocar un lugar. Se eliminaron las políticas muertas del MVC.
- **Bloqueo por usuario en la API**: `AuthService` usa `IsLockedOutAsync`/`AccessFailedAsync`, con los límites por defecto de Identity (5 intentos, 5 minutos), igual que el MVC.
- **Freno por IP** en `POST /api/Auth/login` con el limitador de tasa incluido en ASP.NET Core: 10 intentos por minuto, luego `429`. Complementa al bloqueo por usuario (este frena un ataque contra muchos usuarios; el otro, contra uno).
- **Una sola política de contraseñas** para los dos hosts (`ConfigurarContrasenas`: 8 caracteres, mayúscula, minúscula y número), porque comparten tabla de usuarios. La API usaba la de Identity por defecto, distinta.
- **Cabeceras de seguridad** en los dos hosts: `X-Content-Type-Options`, `X-Frame-Options: DENY`, `Referrer-Policy`.
- **Registro de intentos fallidos** sin correos: se registra el id del usuario o "usuario inexistente".

**Qué NO se hizo (y la condición para hacerlo).**
- **Refresh tokens:** el token dura 8 horas y no se renueva. Es aceptable para una API de uso administrativo; si un cliente interactivo de larga duración la consume, se añade un token de renovación rotativo.
- **CSP (Content-Security-Policy):** las vistas usan scripts en línea y Bootstrap por CDN; una CSP útil exige refactorizarlas. Pendiente.
- **Límite por IP detrás de proxy:** `RemoteIpAddress` sería el del proxy. Al desplegar tras un balanceador hay que usar cabeceras reenviadas (comentado en el código como `shortcut:`).
- **Doble factor (2FA):** Identity lo soporta; no hay requisito aún.

**Alternativas descartadas.**
- Unificar todo en JWT: perdería la sesión por cookie, que es lo más seguro y simple para un sitio renderizado en servidor.
- Un servidor de identidad (OpenIddict/Duende): sobredimensionado para dos aplicaciones y una base de usuarios.
- Autorización por políticas de recursos (por empresa, por registro): hoy el aislamiento entre empresas lo hace el filtro global de EF, ya probado (`MultiempresaTests`).

**En entrevista (30 s).** "Dos hosts, dos mecanismos, cada uno por una razón: cookie para el sitio en servidor, JWT para clientes sin estado. Comparten usuarios, política de contraseñas y políticas de autorización, y la API tiene bloqueo por usuario y freno por IP. Lo que no hice, como refresh tokens y CSP, está escrito con su condición."
