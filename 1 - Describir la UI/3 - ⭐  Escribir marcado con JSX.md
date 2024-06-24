**_JSX_ es una extensión de sintaxis para JavaScript que permite escribir marcado similar a HTML dentro de una archivo JavaScript**. Aunque hay otras formas de escribir componentes, la mayoría de los desarrolladores de React prefieren la concisión de JSX, y la mayoría de las bases de código lo usan.

### Aprenderás

- Por qué React mezcla marcado con lógica de renderizado
- En qué se diferencia JSX de HTML
- Cómo mostrar información con JSX

## JSX: Poniendo marcado dentro de JavaScript

La Web se ha construido sobre HTML, CSS, y JavaScript. Durante muchos años, los desarrolladores web mantuvieron el contenido en HTML, el diseño en CSS, y la lógica en JavaScript, ¡a menudo en archivos separados!. El contenido se marcó dentro del HTML mientras que la lógica de la pagina vivía por separado en JavaScript:

|                    HTML                     |                 JavaScript                  |
| :-----------------------------------------: | :-----------------------------------------: |
| ![[3-escribir-marcado-con-jsx-1.webp\|300]] | ![[3-escribir-marcado-con-jsx-2.webp\|300]] |

Pero, a medida que la Web se volvió más interactiva, la lógica determinó cada vez más el contenido. ¡JavaScript estaba a cargo del HTML! Esto es la razón por la que **en React, la lógica de renderizado y el marcado viven juntos en el mismo lugar: componentes.**

|      Componente de React `Sidebar.js`       |        Componente de React `Form.js`        |
| :-----------------------------------------: | :-----------------------------------------: |
| ![[3-escribir-marcado-con-jsx-3.webp\|300]] | ![[3-escribir-marcado-con-jsx-4.webp\|300]] |

**Mantener juntas la lógica de renderizado y el marcado de un botón, garantiza que permanezcan sincronizados entre sí en cada edición**. Por el contrario, los detalles que no están relacionados, como el marcado de un botón y el marcado de una barra lateral, están aislados entre sí, haciendo que sea más seguro cambiar cualquiera de ellos por su cuenta.

Cada componente de React es una función de JavaScript que puede contener algún marcado que React muestra en el navegador. **Los componentes de React usan una extensión de sintaxis llamada _JSX_ para representar el marcado**. JSX se parece mucho a HTML, pero es un poco más estricto y puede mostrar información dinámica. La mejor manera de comprender esto es convertir algunos marcados HTML en marcado JSX.

> [!note]
>JSX y React son independientes. A menudo se usan en conjunto, pero se _pueden_ [usar de forma separada](https://es.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html#whats-a-jsx-transform). JSX es una extensión de sintaxis, mientras React es una biblioteca de JavaScript.

## Convertir HTML a JSX

Supongamos que tienes algo de HTML (perfectamente válido):

```html
<h1>Tareas Pendientes de Hedy Lamarr</h1>
<img
  src="https://i.imgur.com/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  class="photo"
>
<ul>
    <li>Inventar nuevo semáforo
    <li>Ensayar la escena de la película
    <li>Mejorar la tecnología del espectro
</ul>
```

Y quieres ponerlo en tu componente:

```jsx
export default function TodoList() {
  return (
    // ???
  )
}
```

Si lo copias y pegas tal como está, no funcion3.1ará:

**App.js** ❌
```jsx
export default function TodoList() {
  return ( {/*<- ❌*/}
    // ¡Esto no funciona!
    <h1>Tareas Pendientes de Hedy Lamarr</h1>
    <img 
      src="https://i.imgur.com/yXOvdOSs.jpg" 
      alt="Hedy Lamarr" 
      class="photo"
    > {/*<- ❌*/}
    <ul>
      <li>Inventar nuevo semáforo {/*<- ❌*/}
      <li>Ensayar la escena de la película {/*<- ❌*/}
      <li>Mejorar la tecnología del espectro {/*<- ❌*/}
    </ul>
  );
}
```

Muestrea:

![[3-escribir-marcado-con-jsx-5.png]]

¡**Esto se debe a que JSX es más estricto y tiene algunas restricciones más que HTML**! Si lees los mensajes de error anteriores, te guiarán a corregir el marcado, o puedes seguir la guía a continuación.

> [!note]
>La mayoría de las veces, los mensajes de error en pantalla de React te ayudarán a encontrar donde está el problema. ¡Dales una lectura si te quedas atascado!.

## ⭐ Las Reglas de JSX

### 1. Devolver un solo elemento raíz 

Para devolver múltiples elementos de un componente, **envuélvelos con una sola etiqueta principal**.

Por ejemplo, puedes usar un `<div>`:

```jsx
<div> 👈
  <h1>Tareas Pendientes de Hedy Lamarr</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</div> 👈
```

**Si no deseas agregar un `<div>` adicional a tu marcado, puedes escribir `<>` y `</>` (abreviatura para `React.Fragment`) en su lugar**:

```jsx
<> 👈
  <h1>Tareas Pendientes de Hedy Lamarr</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</> 👈
```

Esta etiqueta vacía se llama un _[Fragmento](https://es.react.dev/reference/react/Fragment)_. Los Fragmentos te permiten agrupar cosas sin dejar ningún rastro en el árbol HTML del navegador.

#### ⭐ ¿Por qué se necesita envolver múltiples etiquetas JSX? [](https://es.react.dev/learn/writing-markup-with-jsx#why-do-multiple-jsx-tags-need-to-be-wrapped "Link for ¿Por qué se necesita envolver múltiples etiquetas JSX?")

JSX parece HTML, pero **por debajo se transforma en objetos planos de JavaScript**. **No puedes devolver dos objetos desde una función sin envolverlos en un array**. Esto explica por qué tampoco puedes devolver dos etiquetas JSX sin envolverlas en otra etiqueta o Fragmento.

### 2. Cierra todas las etiquetas

**JSX requiere que las etiquetas se cierren explícitamente**: las etiquetas de cierre automático como `<img>` deben convertirse en `<img />`, y etiquetas envolventes como `<li>naranjas` deben convertirse como `<li>naranjas</li>`.

Así es como la imagen y los elementos de lista de _Hedy Lamarr_ se ven cerrados:

```jsx
<>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
   /> 👈
  <ul>
    <li>Inventar nuevo semáforo</li> 👈
    <li>Ensayar la escena de la película</li> 👈
    <li>Mejorar la tecnología del espectro</li> 👈
  </ul>
</>
```

### 3. ¡camelCase para ~~todo~~ la mayoría de las cosas!

**JSX se convierte en JavaScript y los atributos escritos en JSX se convierten en _keys_ de objetos JavaScript**. En tus propios componentes, a menudo vas a querer leer esos atributos en variables. Pero JavaScript tiene limitaciones en los nombres de variables. Por ejemplo, sus nombres no pueden contener guiones ni ser palabras reservadas como `class`.

**Por eso, en React, muchos atributos HTML y SVG están escritos en _camelCase_**. Por ejemplo, en lugar de `stroke-width` usa `strokeWidth`. Dado que `class` es una palabra reservada, en React escribes `className` en su lugar, con el nombre de la [propiedad DOM correspondiente](https://developer.mozilla.org/es/docs/Web/API/Element/className):

```jsx
<img 
  src="https://i.imgur.com/yXOvdOSs.jpg" 
  alt="Hedy Lamarr" 
  className="photo" 👈
/>
```

Puedes [encontrar todos estos atributos en la lista de props de los componentes DOM](https://es.react.dev/reference/react-dom/components/common). Si te equivocas en uno, no te preocupes, React imprimirá un mensaje con una posible corrección en la [consola del navegador](https://developer.mozilla.org/docs/Tools/Browser_Console).

> [!warning]
>Por razones históricas, los atributos [`aria-*`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA) y [`data-*`](https://developer.mozilla.org/docs/Learn/HTML/Howto/Use_data_attributes) se escriben como en HTML, con guiones.

### Consejo profesional: usa un convertidor JSX

¡Convertir todos estos atributos en el marcado existente puede ser tedioso! Recomendamos usar un [convertidor](https://transform.tools/html-to-jsx) para traducir su HTML y SVG existente a JSX. Los convertidores son muy útiles en la práctica, pero aun así vale la pena entender lo que sucede así puedes escribir JSX cómodamente por tu cuenta.

**App.js**
```jsx
export default function TodoList() {
  return (
    <>
      <h1>Tareas Pendientes de Hedy Lamarr</h1>
      <img 
        src="https://i.imgur.com/yXOvdOSs.jpg" 
        alt="Hedy Lamarr" 
        className="photo" 
      />
      <ul>
        <li>Inventar nuevo semáforo</li>
        <li>Ensayar la escena de la película</li>
        <li>Mejorar la tecnología del espectro</li>
      </ul>
    </>
  );
}
```
## Recapitulación

Ahora sabes por qué existe JSX y cómo usarlo en componentes:

- Los componentes de React agrupan la lógica de renderización junto con el marcado porque están relacionados.
- JSX es similar a HTML, con algunas diferencias. Puede usar un [convertidor](https://transform.tools/html-to-jsx) si lo necesita.
- Los mensajes de error a menudo te guiarán en la dirección correcta para corregir tu marcado.