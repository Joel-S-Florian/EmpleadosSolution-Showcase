# ADR 010: Respaldos

**Contexto.** Con varias empresas en una sola base de datos ([ADR 009](009-aislamiento-multiempresa.md)) un respaldo es a la vez lo más importante (perder la base es perder a todos los clientes) y lo más peligroso (el archivo contiene salarios y cédulas de todas las empresas).

**Decisión.**
- **Respaldo completo con `BACKUP DATABASE`**, programado: al arrancar si el último tiene más de 24 horas y cada día a las 2:00. Retención de 14 días (`Respaldos:DiasRetencion`); los más viejos se borran al crear uno nuevo. Con `CHECKSUM`.
- **Solo la plataforma** (`SuperAdmin`) lo ve, lo crea a mano y lo descarga. La descarga pide confirmación, es un POST con token antiforgery, solo sirve archivos `.bak` de la carpeta configurada (el nombre llega del navegador y no se usa como ruta) y deja una línea de advertencia en el log con quién y qué archivo.
- **Exportación por empresa en JSON** para su administrador: sus datos (empleados, nóminas, préstamos, incidencias, liquidaciones…) para llevárselos. Usa el filtro global de empresa, así que no puede traer datos de otra, y no incluye usuarios ni contraseñas. Es portabilidad, no respaldo: no hay importación.
- **Desactivado por defecto**: sin `Respaldos:Carpeta` no hay respaldos y la pantalla lo explica. La carpeta debe existir en la máquina del motor de base de datos y ser escribible por él.
- **La restauración se hace con SQL Server**, no desde la aplicación (restaurar la base en uso desde la propia aplicación es peligroso). La pantalla muestra el comando. Una prueba contra SQL Server real crea un respaldo con el servicio y lo restaura en una base nueva con otro nombre, comprobando los datos.

**Qué NO se hizo (y cuándo hacerlo).**
- **Respaldo fuera de la máquina:** los `.bak` quedan en el mismo servidor; un fallo del disco los pierde juntos con la base. Copiarlos a almacenamiento externo (otro servidor, nube) es el siguiente paso antes de producción.
- **Cifrado del archivo:** el `.bak` va sin cifrar. SQL Server permite `WITH ENCRYPTION` con certificado; hacerlo cuando haya dónde guardar el certificado.
- **Respaldo y restauración por empresa:** la exportación no se puede importar. Si un cliente pide restaurar solo lo suyo, hace falta un importador (o restaurar a una base temporal y copiar).
- **Registro duradero:** las ejecuciones y descargas van al log de la aplicación, que se rota; una tabla de auditoría las haría consultables.
- **La prueba de restauración automática** se omite en Linux/contenedores (la carpeta debe ser visible para el motor y para la prueba) y corre en local con LocalDB.

**Alternativas descartadas.** Exportar tablas a archivos desde la aplicación como respaldo general: más lento, sin garantía de consistencia y sin restauración probada. Usar la copia de seguridad del proveedor de la nube: válida en producción, pero no existe en desarrollo y no se puede probar aquí.

**En entrevista (30 s).** "Respaldo nativo de SQL Server, programado y con retención, restringido a la plataforma porque contiene los datos de todos los clientes. Cada descarga se registra. Y lo más importante: una prueba lo restaura en una base limpia, porque un respaldo sin restauración probada no es un respaldo. Lo que falta, como copiarlo fuera del servidor y cifrarlo, está escrito."
