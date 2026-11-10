# ADR 004: Separar la generación de documentos y fijar las fronteras entre proyectos

**Contexto.** `Empleados.Data` mezclaba entidades, `DbContext`, servicios de negocio y generación de PDF, Excel y CSV, con la dependencia de QuestPDF incluida. El nombre del proyecto no decía lo que contenía y cualquier cambio en un PDF recompilaba la capa de datos.

**Decisión.** Estructura final (las flechas son "depende de"):

```
Empleados.Nomina         motor de cálculo puro, sin dependencias del proyecto
Empleados.Data      →    Nomina        entidades, DbContext, servicios de negocio, correo, fotos
Empleados.Documentos →   Data          PDF (QuestPDF), Excel, CSV
Empleados.Api / Mvc →    Data, Documentos
```

- `Data` ya no conoce QuestPDF ni `Documentos`. Lo único que necesita de ellos (el PDF del recibo para el correo) lo pide por `IGeneradorReciboPdf`, definida en `Data` e implementada en `Documentos` (se registra con `AddDocumentos()`).
- Los controladores usan los generadores estáticos directamente: son funciones puras de entrada a bytes, sin estado que simular.
- `Textos` (nombre para mostrar de un enum) se quedó en `Data`, porque lo usan los servicios.
- `ArquitecturaTests` falla si `Data` vuelve a referenciar `Documentos` o QuestPDF, o si el motor de nómina referencia algo del proyecto o EF Core.

**Qué NO se hizo, y por qué (esta es la parte que se defiende).** El plan original proponía además separar Dominio, Aplicación e Infraestructura (5 proyectos). Se descartó por ahora: los servicios de negocio usan `EmpleadosDbContext` directamente (decisión de no envolver EF en repositorios, porque el `DbContext` ya es unidad de trabajo y repositorio). Sin esa abstracción, una capa de aplicación separada tendría que referenciar EF Core y el `DbContext`, y la separación sería solo mover archivos de carpeta sin aislar nada. Separamos lo que sí tiene una frontera real: el motor puro, los documentos y, detrás de interfaces que ya existían, el correo (`IEnviadorCorreo`) y el contexto de usuario.

**Alternativas descartadas.**
- 5 proyectos con `IAppDbContext`: añade una abstracción para satisfacer un diagrama, sin segundo consumidor que la justifique.
- Mover también correo y fotos a otro proyecto: ya están detrás de interfaces; moverlos añade un proyecto más sin quitar acoplamiento.
- Cambiar los espacios de nombres de las entidades: habría obligado a reescribir las migraciones históricas (los nombres de entidad están en el snapshot).

**Consecuencias.** Un proyecto más; `has-pending-model-changes` sigue limpio. Si algún día aparece un segundo almacenamiento o un cliente que no use EF, esa será la razón para dividir aplicación e infraestructura.

**En entrevista (30 s).** "Separé lo que tiene una frontera real: el motor de nómina, que es puro, y la generación de documentos, que es una dependencia pesada. No partí aplicación e infraestructura porque sin repositorios habría sido mover carpetas; lo dejé escrito con la condición para revisarlo."
