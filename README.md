# Laboratorio 1 Segundo Cómputo – Vue.js

## Integrantes
- Yoselin Andrea Linares Hernandez 
- Katerinne Alejandra Mendez Garcia


## Situación problemática

En los pequeños negocios de la zona como tiendas de barrio, ferreterías, papelerías y mini mercados, se enfrenta diariamente a un desafío común: la falta de control sobre el inventario de productos. Los dueños de estos establecimientos suelen llevar el control de sus productos de manera manual (en cuadernos o libretas) o mediante sistemas rudimentarios que no les permiten tener una visión clara de cuántos productos tienen en stock, cuál es el valor total de su inventario, qué productos están próximos a agotarse y qué categorías tienen mayor rotación.

Esta falta de organización provoca pérdidas económicas por productos vencidos, desabastecimiento de productos demandados, sobrecompra de mercadería que no se vende, y dificultad para tomar decisiones de negocio informadas.

Esta situación afecta principalmente al sector del **comercio minorista**, especialmente a **tiendas de barrio, ferreterías, papelerías y pequeños negocios familiares**, donde se requiere una mejor organización para administrar el inventario de manera rápida y clara.

Para resolver este problema, se propone una página web desarrollada con Vue.js que permita registrar productos del inventario, validar los datos ingresados, mostrar una lista de productos activos con sus precios y cantidades, calcular el valor total del stock, filtrar por categoría, identificar productos con bajo stock y eliminar registros cuando ya no sean necesarios. De esta forma se mejora el control, se reduce el error humano y se facilita la gestión de inventario para los comerciantes.



## Funciones del sistema

- Registrar un nuevo producto en el inventario.
- Capturar el nombre del producto.
- Seleccionar la categoría del producto (Alimentos, Bebidas, Limpieza, Electrónica, Papelería).
- Ingresar el precio del producto.
- Ingresar la cantidad en stock.
- Validar campos obligatorios.
- Mostrar mensajes de error si faltan datos o son incorrectos.
- Mostrar mensaje de éxito cuando el producto se guarda correctamente.
- Listar todos los productos registrados.
- Filtrar productos por categoría.
- Calcular automáticamente el valor total del inventario.
- Identificar productos con bajo stock (menos de 5 unidades).
- Mostrar alerta visual para productos con bajo stock.
- Eliminar productos del inventario.

---

PREGUNTAS
## ¿Qué es Vue.js y cuál es su función dentro de la página web desarrollada?

Vue.js es un framework de JavaScript que sirve para crear interfaces web interactivas y dinámicas. En este proyecto se utiliza para manejar los datos de la página en tiempo real, capturar lo que el usuario escribe en los campos del formulario, validar la información y actualizar automáticamente la lista de productos sin necesidad de recargar la página. Vue usa directivas como `v-model`, `v-bind`, `v-for` y `v-if`, además de eventos como `@click` para reaccionar a las acciones del usuario.

---

## Describa qué variables reactivas utilizó en su aplicación y cuál es la función de cada una dentro del sistema.

Las variables reactivas usadas en la aplicación son las siguientes:

| Variable | Función dentro del sistema |
|----------|---------------------------|
| `nuevoProducto` | Almacena los datos del formulario para agregar un nuevo producto (nombre, categoría, precio, cantidad). Permite capturar la información ingresada por el usuario. |
| `productos` | Es el arreglo principal que contiene todos los productos registrados en el inventario. Cada producto tiene id, nombre, categoría, precio y cantidad. |
| `filtroCategoria` | Almacena la categoría seleccionada por el usuario para filtrar la lista de productos. Su valor cambia cuando el usuario elige una opción en el select. |
| `mensajeValidacion` | Guarda mensajes de error o confirmación para mostrar al usuario. Ejemplos: " El nombre del producto es obligatorio" o " Producto agregado exitosamente". |
| `nextId` | Mantiene el siguiente ID disponible para asignar a nuevos productos. Aunque no se muestra directamente en la interfaz, es importante para identificar cada producto de forma única. |

---

## Explique la diferencia entre las siguientes directivas utilizadas en su proyecto: v-bind y v-model

La diferencia es que `v-bind` se usa para enlazar atributos o propiedades de un elemento HTML con datos de Vue. Por ejemplo, en este proyecto se usa para deshabilitar el botón cuando el formulario no es válido (`:disabled="!formularioValido"`) o para cambiar clases CSS condicionalmente (`:class="{ 'warning': productosBajoStock > 0 }"`).

En cambio, `v-model` se usa para enlazar un campo de formulario con una variable reactiva, de manera que lo que el usuario escribe se guarda automáticamente en esa variable. En este proyecto, `v-model` se utiliza en los inputs de nombre, precio, cantidad y en el select de categoría.

En resumen, `v-bind` conecta atributos dinámicos y `v-model` conecta entradas de formulario con los datos del sistema.

---

## Mencione al menos un ejemplo de evento utilizado dentro de su aplicación.

Un evento utilizado fue `@click` en los botones de la aplicación. Este evento se ejecuta cuando el usuario presiona un botón. Por ejemplo:

- `@click="agregarProducto"` → Se ejecuta cuando el usuario hace clic en "Agregar Producto", validando los datos y guardando el producto.
- `@click="eliminarProducto(producto.id)"` → Se ejecuta cuando el usuario hace clic en el botón de eliminar (🗑️) de cada producto, removiéndolo del inventario.

También se utilizó `@click="limpiarFormulario"` para limpiar todos los campos del formulario. Vue permite escuchar eventos del DOM con `v-on` o su forma corta `@`.

---

## Explique para qué utilizó la directiva v-for dentro de su aplicación.

La directiva `v-for` se utiliza para recorrer y mostrar en pantalla la lista de productos registrados. Gracias a esta directiva, cada vez que se agrega un nuevo producto, este aparece automáticamente en la tabla sin necesidad de actualizar manualmente la página.

En este proyecto, `v-for` se usa en dos lugares:

1. **En la tabla de productos**: `<tr v-for="(producto, index) in productosFiltrados" :key="producto.id">` → Recorre todos los productos filtrados y crea una fila por cada uno.
2. **En la alerta de bajo stock**: `<span v-for="producto in productosConBajoStock" :key="producto.id">` → Recorre los productos con menos de 5 unidades y los muestra en la alerta.

---

## Describa en qué situación utilizó v-if y qué problema resuelve dentro de su interfaz.

La directiva `v-if` se utiliza para mostrar mensajes de error, mensajes de éxito y también para indicar cuando no hay productos registrados. Esto ayuda a que la interfaz sea más clara, ya que solo muestra cierta información cuando realmente se necesita. `v-if` se usa para renderizar contenido condicionalmente.

En este proyecto, `v-if` se usa en las siguientes situaciones:

- **Mensajes de validación**: `<p v-if="mensajeValidacion" class="error-message">{{ mensajeValidacion }}</p>` → Solo muestra el mensaje si existe.
- **Estado vacío de la tabla**: `<div v-if="productosFiltrados.length === 0" class="empty-state">` → Si no hay productos, muestra un mensaje amigable; si hay, muestra la tabla.
- **Alerta de bajo stock**: `<div v-if="productosBajoStock > 0" class="alert-section">` → Solo muestra la alerta si existen productos con menos de 5 unidades.

---

## Explique cómo se realiza la validación de datos en su aplicación y por qué es importante validar la información ingresada por el usuario

La validación se realiza de dos formas en la aplicación:

**1. Validación en tiempo real:** A través de una variable computada `formularioValido` que verifica constantemente que todos los campos estén llenos y tengan valores válidos, deshabilitando el botón de agregar si no se cumplen las condiciones.

**2. Validación al momento de agregar:** Se revisa que:
- El nombre no esté vacío
- Se haya seleccionado una categoría
- El precio sea mayor a 0
- La cantidad no sea negativa

Si algún dato es incorrecto, el sistema muestra un mensaje de error y no permite guardar el producto.

**Validar los datos es importante porque:**
- Evita registros incompletos o incorrectos
- Mejora la confiabilidad del sistema
- Ayuda a que la información almacenada sea útil y ordenada
- Previene cálculos erróneos como el valor total del stock
- Mejora la experiencia del usuario al mostrarle exactamente qué debe corregir

---

## Tecnologías utilizadas

- **HTML5** → Estructura de la aplicación
- **CSS3** → Estilos y diseño responsivo
- **JavaScript** → Lógica de programación
- **Vue.js 3** → Framework para la interfaz reactiva
- **Vite** → Herramienta de desarrollo y empaquetado

---

## Etiquetas HTML utilizadas

En el proyecto utilizamos más de cinco etiquetas HTML, por ejemplo:

| Etiqueta | Uso |
|----------|-----|
| `<header>` | Encabezado de la aplicación |
| `<div>` | Contenedores para organización |
| `<h1>`, `<h2>` | Títulos y subtítulos |
| `<form>` | No se usó, pero se utilizó `<div class="form-section">` para simular formulario |
| `<input>` | Campos para nombre, precio y cantidad |
| `<select>` | Selección de categoría |
| `<button>` | Botones para agregar, limpiar y eliminar |
| `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>` | Tabla para listar productos |
| `<p>` | Mensajes de validación y textos |
| `<span>` | Badges de categoría y productos en alerta |

---

## Directivas Vue utilizadas

| Directiva | Uso en el proyecto |
|-----------|-------------------|
| `v-model` | Enlaza los inputs del formulario con las variables reactivas (nombre, categoría, precio, cantidad) |
| `v-bind` (`:`) | Deshabilita el botón condicionalmente y aplica clases CSS dinámicas |
| `v-for` | Recorre la lista de productos para mostrarlos en la tabla y en la alerta de bajo stock |
| `v-if` | Muestra condicionalmente mensajes de validación, estado vacío y alertas |
