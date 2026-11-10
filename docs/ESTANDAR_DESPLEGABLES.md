# Estándar de desplegables (Smart Select)

Todos los desplegables del sistema usan el mismo componente: `wwwroot/js/smart-select.js` (estilos en `wwwroot/css/site.css`, bloque «Smart Select»). Se carga en `_Layout.cshtml` y se aplica solo.

## Cómo se usa

Se escribe un `<select>` normal de Razor con la clase `form-select`:

```cshtml
<label for="EmpleadoId" class="app-etiqueta">Empleado</label>
<select id="EmpleadoId" name="EmpleadoId" asp-items="ViewBag.Empleados" class="form-select" required>
    <option value="">Seleccione…</option>
</select>
```

No hace falta nada más. El componente:

- Dibuja un botón con el mismo aspecto que un `form-select` (borde redondeado y flecha) y, al hacer clic, abre su propio menú.
- Agrega un **buscador** arriba cuando hay **9 opciones o más**. Busca sin distinguir tildes ni mayúsculas.
- Muestra solo las **primeras 8 opciones**, y carga 8 más al llegar al final de la lista o al buscar. El pie dice cuántas se están mostrando.
- Se **cierra** al elegir una opción, al hacer clic fuera, con `Escape` o con `Tab`.
- Dibuja el menú **por encima de todo**, en `<body>` y con posición fija. Ninguna tarjeta, tabla ni contenedor con `overflow` lo recorta. Si abajo no cabe, se abre hacia arriba, y sigue al botón cuando la página hace scroll.
- Se maneja con el **teclado**: flechas para moverse, `Enter` para elegir y `Escape` para cerrar.

## Reglas

1. **El `<select>` original se queda en el formulario.** El envío, `asp-for`, la validación (`required`) y los eventos `change`, incluidos los `onchange` en línea como `onchange="this.form.submit()"`, funcionan igual. No se lee ni se escribe el botón: siempre el `<select>`.
2. **La opción vacía va primero** (`value=""`: «Seleccione…», «Todos», «Toda la empresa»). Siempre se muestra arriba cuando no se busca.
3. **Orden: lo más usado o lo más reciente primero.** Como al abrir solo se ven 8, el servidor ordena la lista para que esas 8 sean las útiles. Por ejemplo, los empleados van del ingreso más reciente al más antiguo (`EmpleadoService.ParaSeleccionAsync`) y las nóminas, de la más nueva a la más vieja.
4. **Listas completas.** No se cortan las opciones en el servidor (por ejemplo, con `pageSize`): la búsqueda y la carga progresiva ya evitan dibujar miles de elementos.
5. **Etiqueta.** Cada desplegable tiene su `<label for="id">` (o un `aria-label`). El componente la pasa al botón.
6. **Opciones que cambian por JavaScript** (como los puestos según el departamento). Basta con modificar las `<option>` del `<select>`: el componente lo detecta solo.
7. **Contenido que se agrega después** (HTML cargado con `fetch`). Hay que llamar a `SmartSelect.iniciar(contenedor)`.
8. **Excepciones.** `<select multiple>` no se transforma. Para dejar un `<select>` nativo a propósito se usa `data-nativo`, y hay que justificarlo en un comentario.

## Cuándo no usar un desplegable

- Con 2 o 3 opciones fijas que conviene ver de una vez (sí/no, quincena 1/2), son mejores los *radio buttons* o un interruptor.
- Para buscar entre miles de registros del servidor (por ejemplo, todos los recibos históricos), el desplegable no sirve: hace falta un buscador con resultados paginados.
