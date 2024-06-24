Tu aplicación de React está tomando forma con muchos componentes anidados entre sí. ¿Cómo hace React para hacer un seguimiento de la estructura de componentes de tu aplicación?

React, al igual que muchas otras bibliotecas de UI, modela la UI como un árbol. Pensar en tu aplicación como un árbol es útil para comprender la relación entre los componentes. Esta comprensión te ayudará a depurar conceptos futuros como el rendimiento y la gestión del estado.

### Aprenderás

- Cómo «ve» React las estructuras de componentes
- Qué es un árbol de renderizado y para qué es útil
- Qué es un árbol de dependencias de módulos y para qué es útil

## Tu UI como un árbol

Los árboles son un modelo de relación entre elementos, y la UI se representa frecuentemente utilizando estructuras de árbol. Por ejemplo, los navegadores utilizan estructuras de árbol para modelar HTML ([DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction)) y CSS ([CSSOM](https://developer.mozilla.org/docs/Web/API/CSS_Object_Model)). Las plataformas móviles también utilizan árboles para representar su jerarquía de vistas.

![Diagrama con tres secciones dispuestas horizontalmente. En la primera sección, hay tres rectángulos apilados verticalmente, con las etiquetas 'Componente A', 'Componente B' y 'Componente C'. Transicionando a la siguiente sección hay una flecha con el logo de React en la parte superior etiquetada 'React'. La sección del medio contiene un árbol de componentes, con la raíz etiquetada 'A' y dos hijos etiquetados 'B' y 'C'. La siguiente sección se transiciona nuevamente usando una flecha con el logo de React en la parte superior etiquetada 'React DOM'. La tercera y última sección es un esquema de un navegador, que contiene un árbol de 8 nodos, de los cuales solo un subconjunto está resaltado (indicando el subárbol de la sección del medio).](https://es.react.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_dom_tree.dark.png&w=1920&q=75)

React crea un árbol de UI a partir de tus componentes. En este ejemplo, el árbol de UI se utiliza luego para renderizar en el DOM.

Al igual que los navegadores y las plataformas móviles, React también utiliza estructuras de árbol para gestionar y modelar la relación entre los componentes en una aplicación de React. Estos árboles son herramientas útiles para comprender cómo fluye la información a través de una aplicación de React y cómo optimizar el renderizado y el tamaño de la aplicación.

## El árbol de renderizado

Una característica importante de los componentes es la capacidad de componer componentes de otros componentes. **Al [anidar componentes](https://es.react.dev/learn/your-first-component#nesting-and-organizing-components), tenemos el concepto de _componentes padre e hijo_, donde cada componente padre puede ser a su vez un hijo de otro componente**.

Cuando renderizamos una aplicación de React, podemos modelar esta relación en un árbol, conocido como el árbol de renderizado.

Aquí hay una aplicación de React que renderiza citas inspiradoras.

**App.js**
```jsx
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}
```

**FancyText.js**
```jsx
export default function FancyText({title, text}) {
  return title
    ? <h1 className='fancy title'>{text}</h1>
    : <h3 className='fancy cursive'>{text}</h3>
}
```

**InspirationGenerator.js**
```jsx
import * as React from 'react';
import quotes from './quotes';
import FancyText from './FancyText';

export default function InspirationGenerator({children}) {
  const [index, setIndex] = React.useState(0);
  const quote = quotes[index];
  const next = () => setIndex((index + 1) % quotes.length);

  return (
    <>
      <p>Your inspirational quote is:</p>
      <FancyText text={quote} />
      <button onClick={next}>Inspire me again</button>
      {children}
    </>
  );
```

**Copyright.js**
```jsx
export default function Copyright({year}) {
  return <p className='small'>©️ {year}</p>;
}
```

**quotes.js**
```jsx
export default [
  "Don’t let yesterday take up too much of today.” — Will Rogers",
  "Ambition is putting a ladder against the sky.",
  "A joy that's shared is a joy made double.",
  ];
```
Muestra:

![[9-entender-la-ui-com-un-arbol-2.png]]


![[9-entender-la-ui-com-un-arbol-1.webp]]

>React crea un _árbol de renderizado_, un árbol de interfaz de usuario, compuesto por los componentes renderizados.

A partir de la aplicación de ejemplo, podemos construir el árbol de renderizado anterior.

El árbol está compuesto por nodos, cada uno de los cuales representa un componente. `App`, `FancyText`, `Copyright`, por nombrar algunos, son todos nodos en nuestro árbol.

El nodo raíz en un árbol de renderizado de React es el [componente raíz](https://es.react.dev/learn/importing-and-exporting-components#the-root-component-file) de la aplicación. En este caso, el componente raíz es `App` y es el primer componente que React renderiza. Cada flecha en el árbol apunta desde un componente padre hacia un componente hijo.

>[!info]
>#### ¿Dónde están las etiquetas HTML en el árbol de renderizado?
>
>Notarás en el árbol de renderizado anterior que no se mencionan las etiquetas HTML que cada componente renderiza. Esto se debe a que **el árbol de renderizado está compuesto únicamente por [componentes de React](https://es.react.dev/learn/your-first-component#components-ui-building-blocks)**.
>
>React, como framework de UI, es independiente de la plataforma. En _react.dev_, mostramos ejemplos que se renderizan en la web, la cual utiliza marcado HTML como sus primitivas de UI. Sin embargo, una aplicación de React también podría renderizarse en una plataforma móvil o de escritorio, que podría utilizar diferentes primitivas de UI como [UIView](https://developer.apple.com/documentation/uikit/uiview) o [FrameworkElement](https://learn.microsoft.com/en-us/dotnet/api/system.windows.frameworkelement?view=windowsdesktop-7.0).
>
>Estas primitivas de UI de plataforma no son parte de React. Los árboles de renderizado de React pueden proporcionar información sobre nuestra aplicación de React independientemente de la plataforma en la que se renderice tu aplicación.

Un árbol de renderizado representa un único pase de renderizado de una aplicación de React. Con [renderizado condicional](https://es.react.dev/learn/conditional-rendering), un componente padre puede renderizar diferentes hijos dependiendo de los datos pasados.

Podemos actualizar la aplicación para renderizar condicionalmente una cita inspiradora o un color.


**App.js**
```jsx
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}
```

**FancyText.js**
```jsx
export default function FancyText({title, text}) {
  return title
    ? <h1 className='fancy title'>{text}</h1>
    : <h3 className='fancy cursive'>{text}</h3>
}
```

**InspirationGenerator.js**
```jsx
import * as React from 'react';
import quotes from './quotes';
import FancyText from './FancyText';

export default function InspirationGenerator({children}) {
  const [index, setIndex] = React.useState(0);
  const quote = quotes[index];
  const next = () => setIndex((index + 1) % quotes.length);

  return (
    <>
      <p>Your inspirational quote is:</p>
      <FancyText text={quote} />
      <button onClick={next}>Inspire me again</button>
      {children}
    </>
  );
```

**Copyright.js**
```jsx
export default function Copyright({year}) {
  return <p className='small'>©️ {year}</p>;
}
```

**quotes.js**
```jsx
export default [
  "Don’t let yesterday take up too much of today.” — Will Rogers",
  "Ambition is putting a ladder against the sky.",
  "A joy that's shared is a joy made double.",
  ];
```

Muestra:

![[9-entender-la-ui-com-un-arbol-2.png]]

![[9-entender-la-ui-com-un-arbol-3.png]]

  
![[9-entender-la-ui-com-un-arbol-4.webp]]

>Con el renderizado condicional, en diferentes renderizados, el árbol de renderizado puede renderizar diferentes componentes.

En este ejemplo, dependiendo de lo que sea `inspiration.type`, podemos renderizar `<FancyText>` o `<Color>`. El árbol de renderizado puede ser diferente para cada pase de renderizado.

Aunque los árboles de renderizado pueden diferir en cada pase de renderizado, estos árboles son generalmente útiles para identificar cuáles son los _componentes de nivel superior_ y _componentes hoja_ en una aplicación de React. Los componentes de nivel superior son los componentes más cercanos al componente raíz y afectan al rendimiento de renderizado de todos los componentes debajo de ellos y a menudo contienen la mayor complejidad. Los componentes hoja están cerca del final del árbol y no tienen componentes hijos y a menudo se vuelven a renderizar con frecuencia.

Identificar estas categorías de componentes es útil para comprender el flujo de datos y el rendimiento de tu aplicación.

## El árbol de dependencias de módulos

**Otra relación en una aplicación de React que se puede modelar con un árbol son las _dependencias_ de módulos de una aplicación**. A medida que [dividimos nuestros componentes](https://es.react.dev/learn/importing-and-exporting-components#exporting-and-importing-a-component) y lógica en archivos separados, creamos [módulos de JS](https://developer.mozilla.org/es/docs/Web/JavaScript/Guide/Modules) donde podemos exportar componentes, funciones o constantes.

Cada nodo en un árbol de dependencias de módulos es un módulo y cada rama representa una declaración `import` en ese módulo.

Si tomamos la aplicación de Inspiraciones anterior, podemos construir un árbol de dependencias de módulos, o de forma abreviada, árbol de dependencias.

![[9-entender-la-ui-com-un-arbol-5.webp]]

>El árbol de dependencias de módulos para la aplicación de Inspiraciones.

El nodo raíz del árbol es el módulo raíz, también conocido como archivo de entrada. A menudo es el módulo que contiene el componente raíz.

Comparando con el árbol de renderizado de la misma aplicación, hay estructuras similares pero algunas diferencias notables:

- Los nodos que conforman el árbol representan módulos, no componentes.
- Los módulos que no son componentes, como `inspirations.js`, también están representados en este árbol. El árbol de renderizado solo encapsula componentes.
- `Copyright.js` aparece bajo `App.js`, pero en el árbol de renderizado, `Copyright`, el componente, aparece como hijo de `InspirationGenerator`. Esto se debe a que `InspirationGenerator` acepta JSX como [props hijas](https://es.react.dev/learn/passing-props-to-a-component#passing-jsx-as-children), por lo que renderiza `Copyright` como un componente hijo pero no importa el módulo.

**Los árboles de dependencias son útiles para determinar qué módulos son necesarios para ejecutar tu aplicación de React**. Al construir una aplicación de React para producción, típicamente hay un paso de construcción que empaquetará todo el JavaScript necesario para enviar al cliente. La herramienta responsable de esto se llama un [empaquetador](https://developer.mozilla.org/es/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Overview#the_modern_tooling_ecosystem), y **los empaquetadores utilizarán el árbol de dependencias para determinar qué módulos deben incluirse**.

A medida que tu aplicación crece, a menudo también lo hace el tamaño del paquete. Los tamaños de paquete grandes son costosos para un cliente descargar y ejecutar. Los tamaños de paquete grandes pueden retrasar el tiempo para que se dibuje tu interfaz de usuario. Tener una idea del árbol de dependencias de tu aplicación puede ayudar a depurar estos problemas.

## Recapitulación

- Los árboles son una forma común de representar la relación entre entidades. A menudo se utilizan para modelar la interfaz de usuario.
- Los árboles de renderizado representan la relación anidada entre los componentes de React en un solo renderizado.
- Con el renderizado condicional, el árbol de renderizado puede cambiar en diferentes renderizados. Con diferentes valores de propiedades, los componentes pueden renderizar diferentes componentes hijos.
- Los árboles de renderizado ayudan a identificar cuáles son los componentes de nivel superior y hoja. Los componentes de nivel superior afectan al rendimiento de renderizado de todos los componentes debajo de ellos y los componentes hoja a menudo se vuelven a renderizar con frecuencia. Identificarlos es útil para comprender y depurar el rendimiento de renderizado.
- Los árboles de dependencias representan las dependencias de módulos en una aplicación de React.
- Las herramientas de construcción utilizan los árboles de dependencias para empaquetar el código necesario para publicar una aplicación.
- Los árboles de dependencias son útiles para depurar tamaños de paquete grandes que ralentizan el tiempo de pintado y exponen oportunidades para optimizar qué código se empaqueta.