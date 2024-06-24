> https://es.react.dev/learn/your-first-component

 # Tu primer componente

Los _componentes_ son uno de los conceptos esenciales de React. Constituyen los cimientos sobre los que construyes interfaces de usuario (UIs por sus siglas en inglés). ¡Y eso los convierte en el lugar perfecto para comenzar tu recorrido por React!

### Aprenderás

- Qué es un componente
- Qué papel desempeñan los componentes en una aplicación de React
- Cómo escribir tu primer componente de React

## Componentes: Elementos básicos para construir _UIs_ 

En la Web, HTML nos permite crear documentos estructurados con su conjunto integrado de etiquetas como `<h1>` y `<li>`:

```html
<article>
  <h1>Mi primer componente</h1>
  <ol>
    <li>Componentes: Bloques de construcción de la UI</li>
    <li>Definiendo un componente</li>
    <li>Usando un componente</li>
  </ol>
</article>
```

Este marcado representa un artículo `<article>`, su encabezado `<h1>`, y una (abreviada) tabla de contenidos representada como una lista ordenada `<ol>`. Un marcado como este, combinado con CSS para los estilos y JavaScript para la interactividad, están detrás de cada barra lateral, avatar, modal, menú desplegable y cualquier otra pieza de UI que ves en la web.

React te permite combinar tu marcado, CSS y JavaScript en «_componentes_» personalizados, **elementos reutilizables de UI para tu aplicación.** El código de la tabla de contenidos que viste arriba pudo haberse transformado en un componente `<TableOfContents />` que podrías renderizar en cada página. Por detrás, seguiría utilizando las mismas etiquetas HTML como `<article>`, `<h1>`, etc.

De la misma forma que con las etiquetas HTML, puedes componer, ordenar y anidar componentes para diseñar páginas completas. Por ejemplo la página de documentación que estás leyendo está hecha de componentes de React:

```jsx
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Documentación</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

En la medida en que tu proyecto crece, notarás que muchos de tus diseños se pueden componer mediante la reutilización de componentes que ya escribiste, acelerando el desarrollo. ¡Nuestra tabla de contenido de arriba podría añadirse a cualquier página con `<TableOfContents />`! Incluso puedes montar tu proyecto con los miles de componentes compartidos por la comunidad de código abierto de React como [Chakra UI](https://chakra-ui.com/) y [Material UI](https://material-ui.com/).

## Definir un componente

Tradicionalmente, cuando se creaban páginas web, los desarrolladores web usaban lenguaje de marcado para describir el contenido y luego añadían interacción agregando un poco de JavaScript. Esto funcionaba perfectamente cuando las interacciones eran algo _deseable, pero no imprescindible_ en la web. Ahora es algo que se espera de muchos sitios y de todas las aplicaciones. React pone la interactividad primero usando aún la misma tecnología: **un componente de React es una función de JavaScript a la que puedes _agregar marcado_**. Aquí vemos cómo luce esto (puede editar el ejemplo de abajo):

**App.jsx**
```jsx
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

Y aquí veremos cómo construir un componente:

![[1-tu-primer-componente-1.png]]

### Paso 1: Exporta el componente 

El prefijo `export default` es parte de la sintaxis [estándar de Javascript](https://developer.mozilla.org/docs/web/javascript/reference/statements/export) (no es específico de React). Te permite marcar la función principal en un archivo para que luego puedas importarlas en otros archivos. (¡Más sobre importar en [Importar y exportar componentes](https://es.react.dev/learn/importing-and-exporting-components)!).

### Paso 2: Define la función

Con `function Profile() { }` defines una función con el nombre `Profile`.

> [!warning]
> ¡Los componentes de React son funciones regulares de JavaScript, pero **sus nombres deben comenzar con letra mayúscula** o no funcionarán!

### ⭐ Paso 3: Agrega marcado 

El componente devuelve una etiqueta `<img />` con atributos `src` y `alt`. `<img />` se escribe como en HTML, ¡pero en realidad es JavaScript por detrás! Esta sintaxis se llama [JSX](https://es.react.dev/learn/writing-markup-with-jsx), y te permite incorporar marcado dentro de JavaScript.

**Las sentencias `return` se pueden escribir todo en una línea**, como en este componente:

```tsx
return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
```

**Pero si tu marcado no está todo en la misma línea que la palabra clave `return`, debes ponerlo dentro de paréntesis** como en este ejemplo:

```jsx
return (
  <div>
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  </div>
);
```

> [!warning]
> ¡Sin paréntesis, todo el código que está en las líneas posteriores al `return` [serán ignoradas](https://stackoverflow.com/questions/2846283/what-are-the-rules-for-javascripts-automatic-semicolon-insertion-asi)!

## Usar un componente 

Ahora que has definido tu componente `Profile`, puedes anidarlo dentro de otros componentes. Por ejemplo, puedes exportar un componente `Gallery` que utilice múltiples componentes `Profile`:

**App.js**
```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Científicos increíbles</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```
Muestra:

![[1-tu-primer-componente-2.png]]
### Lo que ve el navegador 

**Nota la diferencia de mayúsculas y minúsculas**:

- `<section>` está en minúsculas, por lo que React sabe que nos referimos a una etiqueta HTML.
- `<Profile />` comienza con una `P` mayúscula, por lo que React sabe que queremos usar nuestro componente llamado `Profile`.

Y `Profile` contiene aún más HTML: `<img />`. Al final lo que el navegador ve es esto:

```jsx
<section>
  <h1>Científicos increíbles</h1>
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
</section>
```

### ⭐ Anidar y organizar componentes 

Los componentes son funciones regulares de JavaScript, por lo que **puedes tener múltiples componentes en el mismo archivo. Esto es conveniente sólo cuando los componentes son relativamente pequeños o están estrechamente relacionados entre sí. Si este archivo se torna abarrotado, siempre puedes mover `Profile` a un archivo separado**. Aprenderás como hacer esto pronto en la [página sobre _importaciones_](https://es.react.dev/learn/importing-and-exporting-components).

Dado que los componentes `Profile` se renderizan dentro de `Gallery` (¡incluso varias veces!) podemos decir que `Gallery` es un **componente padre**, que renderiza cada `Profile` como un «hijo». Este es la parte mágica de React: puedes definir un componente una vez, y luego usarlo en muchos lugares y tantas veces como quieras.

> [!warning]
> Los componentes pueden renderizar otros componentes, pero **nunca debes anidar sus definiciones:**
> 
> ```tsx
export default function Gallery() {
  // 🔴 ¡Nunca definas un componente dentro de otro componente!
>  function Profile() {
>    // ...
>  }
>  // ...
>}
>```
>
>El fragmento de código de arriba es [muy lento y causa errores.](https://es.react.dev/learn/preserving-and-resetting-state#different-components-at-the-same-position-reset-state) En su lugar, define cada componente en el primer nivel:
>
>```jsx
export default function Gallery() {
  // ...
}
>
>// ✅ Declara los componentes en el primer nivel
>function Profile() {
>  // ...
>}
>```

Cuando un componente hijo necesita datos de su padre, [pásalo por props](https://es.react.dev/learn/passing-props-to-a-component) en lugar de anidar las definiciones.

#### Componentes de arriba a abajo

Tu aplicación React comienza en un componente _«root»_ (raíz). Normalmente, se crea automáticamente al iniciar un nuevo proyecto. Por ejemplo, si usas [CodeSandbox](https://codesandbox.io/) o si usas el framework [Next.js](https://nextjs.org/), el componente _root_ se define en `pages/index.js`. En estos ejemplos, has estado exportando componentes _root_.

La mayoría de las aplicaciones React usan componentes _root_. Esto significa que no solo usarás componentes para piezas reutilizables como botones, sino también para piezas más grandes como barras laterales, listas y, en última instancia, ¡páginas completas! Los componentes son una forma práctica de organizar el código de UI y el marcado, incluso si algunos de ellos solo se utilicen una vez.

[Los frameworks basados en React](https://es.react.dev/learn/start-a-new-react-project) llevan esto un paso más allá. En lugar de utilizar un archivo HTML vacío y dejar que React «se encargue» de gestionar la página con JavaScript, _también_ generan el HTML automáticamente a partir de tus componentes React. Esto permite que tu aplicación muestre algún contenido antes de que se cargue el código JavaScript.

Aún así, muchos sitios web sólo usan React para [añadir interactividad a páginas HTML existentes](https://es.react.dev/learn/add-react-to-an-existing-project#using-react-for-a-part-of-your-existing-page). Tienen muchos componentes _root_ en lugar de uno solo para toda la página. Puedes utilizar la cantidad de React que necesites.

## Recapitulación

¡Acabas de probar por primera vez React! Recapitulemos algunos puntos clave.

- React te permite crear componentes, **elementos reutilizables de UI para tu aplicación.**
    
- En una aplicación de React, cada pieza de UI es un componente.
    
- Los componentes de React son funciones regulares de JavaScript excepto que:
    
    1. Sus nombres siempre empiezan con mayúscula.
    2. Devuelven marcado JSX.