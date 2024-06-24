Los componentes a menudo necesitan cambiar lo que se muestra en pantalla como resultado de una interacción. Escribir dentro de un formulario debería actualizar el campo de texto, hacer clic en «_siguiente_» en un carrusel de imágenes debería cambiar la imagen que es mostrada; hacer clic en un botón para comprar un producto debería actualizar el carrito de compras. 

**En los ejemplos anteriores los componentes deben «_recordar_» cosas: el campo de texto, la imagen actual, el carrito de compras. En React, a este tipo de memoria de los componentes se le conoce como _estado_**.

### Aprenderás

- Cómo agregar una variable de estado con el Hook de [`useState`](https://es.react.dev/reference/react/useState)
- Qué par de valores devuelve el hook de `useState`
- Cómo agregar más de una variable de estado
- Por qué se le llama local al estado

## ⭐ Cuando una variable regular no es suficiente

Aquí hay un componente que renderiza una imagen de una escultura. Al hacer clic en el botón «_Siguiente_», debería mostrarse la siguiente escultura cambiando el índice `index` a `1`, luego a `2`, y así sucesivamente. Sin embargo, esto **no funcionará** (¡puedes intentarlo!):

**App.js**
```jsx
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Siguiente
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        por {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} de {sculptureList.length})
      </h3>
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}
```

**data.js**
```jsx
export const sculptureList = [{
  name: 'Homenaje a la neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Aunque Colvin es predominantemente conocida por temas abstractos que aluden a símbolos prehispánicos, esta gigantesca escultura, un homenaje a la neurocirugía, es una de sus obras de arte público más reconocibles.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Una estatua de bronce de dos manos cruzadas sosteniendo delicadamente un cerebro humano en la punta de sus dedos.'
}, {
  name: 'Floralis genérica',
  artist: 'Eduardo Catalano',
  description: 'Esta enorme flor plateada (75 pies o 23 m) se encuentra en Buenos Aires. Está diseñado para moverse, cerrando sus pétalos por la tarde o cuando soplan fuertes vientos y abriéndolos por la mañana.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Una gigantesca escultura de flor metálica con pétalos reflectantes como espejos y fuertes estambres.'
}, {
  name: 'Presencia eterna',
  artist: 'John Woodrow Wilson',
  description: 'Wilson era conocido por su preocupación por la igualdad, la justicia social, así como por las cualidades esenciales y espirituales de la humanidad. Este bronce masivo (7 pies o 2,13 m) representa lo que él describió como "una presencia negra simbólica infundida con un sentido de humanidad universal"."',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'La escultura que representa una cabeza humana parece omnipresente y solemne. Irradia calma y serenidad.'
}];
```

El manejador de evento `handleClick` está actualizando una variable local, `index`. Pero dos cosas impiden que ese cambio sea visible:

1. **Las variables locales no persisten entre renderizaciones.** Cuando React renderiza este componente por segunda vez, lo hace desde cero, no considera ningún cambio en las variables locales.
2. **Los cambios en las variables locales no activarán renderizaciones.** React no se da cuenta de que necesita renderizar el componente nuevamente con los nuevos datos.

Para actualizar un componente con datos nuevos, deben pasar las siguientes dos cosas:

1. **Conservar** los datos entre renderizaciones.
2. **Provocar** que React renderice el componente con nuevos datos (re-renderizado).

El Hook de [`useState`](https://es.react.dev/reference/react/useState) ofrece dos cosas:

1. Una **variable de estado** para mantener los datos entre renderizados.
2. Una **función que setea el estado** para actualizar la variable y provocar que React renderice el componente nuevamente.

## ⭐ Agregar una variable de estado

Para agregar una variable de estado, debemos importar el `useState` de React al inicio del archivo:

```jsx
import { useState } from 'react';
```

Luego, reemplazamos esta línea:

```jsx
let index = 0;
```

con:

```jsx
const [index, setIndex] = useState(0);
```

**`index` es una variable de estado y `setIndex` es la función que setea el estado**.

> [!note]
> La sintaxis de `[` y `]` se le conoce cómo [desestructuración de un array](https://javascript.info/destructuring-assignment) y permite leer valores de un array. El array devuelto por `useState` siempre contará con exactamente dos valores.

Así es como funcionan juntos en el `handleClick`:

```jsx
function handleClick() {
  setIndex(index + 1);
}
```

Ahora al hacer clic en el botón «_Siguiente_» cambia la escultura actual:

**App.js**
```jsx
import { useState } from 'react'; // 👈
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0); // 👈

  function handleClick() {
    setIndex(index + 1); // 👈
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Siguiente
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        por {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} de {sculptureList.length})
      </h3>
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}

```

**data.js**
```jsx
export const sculptureList = [{
  name: 'Homenaje a la neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Aunque Colvin es predominantemente conocida por temas abstractos que aluden a símbolos prehispánicos, esta gigantesca escultura, un homenaje a la neurocirugía, es una de sus obras de arte público más reconocibles.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Una estatua de bronce de dos manos cruzadas sosteniendo delicadamente un cerebro humano en la punta de sus dedos.'
}, {
  name: 'Floralis genérica',
  artist: 'Eduardo Catalano',
  description: 'Esta enorme flor plateada (75 pies o 23 m) se encuentra en Buenos Aires. Está diseñado para moverse, cerrando sus pétalos por la tarde o cuando soplan fuertes vientos y abriéndolos por la mañana.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Una gigantesca escultura de flor metálica con pétalos reflectantes como espejos y fuertes estambres.'
}, {
  name: 'Presencia eterna',
  artist: 'John Woodrow Wilson',
  description: 'Wilson era conocido por su preocupación por la igualdad, la justicia social, así como por las cualidades esenciales y espirituales de la humanidad. Este bronce masivo (7 pies o 2,13 m) representa lo que él describió como "una presencia negra simbólica infundida con un sentido de humanidad universal"."',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'La escultura que representa una cabeza humana parece omnipresente y solemne. Irradia calma y serenidad.'
}];
```

### ⭐ Conoce tu primer Hook

En React, **`useState`, así como cualquier otra función que empiece con `"use"`, se le conoce como Hook**.

**Los _Hooks_ son funciones especiales que sólo están disponibles mientras React está [renderizando](https://es.react.dev/learn/render-and-commit#step-1-trigger-a-render)** (algo que veremos con más detalle en la página siguiente). Los Hooks permiten «_engancharnos_» a diferentes características de React.

El estado es solo una de esas características, pero conoceremos los otros Hooks más adelante.

>[!important] 
> #### ⭐ Hooks sólo deben ser llamadas en un nivel superior
> **Las funciones-Hook que empiecen con `use` deben ser solo llamadas en el nivel superior de los componentes o [en tus propios Hooks.](https://es.react.dev/learn/reusing-logic-with-custom-hooks)**
> 
>**No podemos usar Hooks dentro de condicionales, bucles, u otras funciones anidadas**. Los Hooks son funciones, pero es útil pensar en ellos como declaraciones independientes de las necesidades de nuestro componente. Las funciones de React se «usan» en la parte superior del componente de manera similar a cómo se «importan» módulos en la parte superior de un archivo.

### ⭐ Anatomía del `useState`: Cómo funciona

**Cuando llamamos al [`useState`](https://es.react.dev/reference/react/useState), le estamos diciendo a React que debe recordar algo**:

```jsx
const [index, setIndex] = useState(0);
```

En este caso, queremos que React recuerde el `index`.

> [!tip]
> La convención es nombrar estas dos variables como `const [algo, setAlgo]`. Podemos nombrarlo como queramos, pero mantener las convenciones hacen que las cosas sean más fáciles de entender en todos los proyectos.

El único argumento para `useState` es el **valor inicial** de su variable de estado. En este ejemplo, el valor inicial de `index` se establece en `0` con `useState(0)`.

Cada vez que el componente se renderiza, el `useState` devuelve un _array_ que contiene dos valores:

1. La **variable de estado** (`index`) con el valor que almacenaste.
2. La **función que establece el estado** (`setIndex`) que puede actualizar la variable de estado y alertar a React para que renderice el componente nuevamente.

Así es como sucede en acción:

```JSX
const [index, setIndex] = useState(0);
```

1. **Tu componente se renderiza por primera vez.** Debido a que pasamos `0` a `useState` como valor inicial para `index`, esto devolverá `[0, setIndex]`. React recuerda que `0` es el último valor de estado.
2. **Actualizas el estado.** Cuando un usuario hace clic en el botón, llama a `setIndex(index + 1)`. `index` es `0`, por lo tanto es `setIndex(1)`. Esto le dice a React que recuerde que `index` es `1` ahora y ejecuta otro renderizado.
3. **El componente se renderiza por segunda vez.** React todavía ve `useState(0)`, pero debido a que React _recuerda_ que estableció `index` en `1`, devuelve `[1, setIndex]` en su lugar.
4. ¡Y así sucesivamente!

## Colocar múltiples variables de estado a un componente

Podemos tener más de una variable de estado de diferentes tipos en un componente. Este componente tiene dos variables de estado, un `index` numérico y un `showMore` booleano que se activa al hacer clic en «_Mostrar detalles_»:


**App.js**
```jsx
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0); // 👈
  const [showMore, setShowMore] = useState(false); // 👈

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>
        Siguiente
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        por {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} de {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? 'Ocultar' : 'Mostrar'} detalles
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
    </>
  );
}
```

**data.js**
```jsx
export const sculptureList = [{
  name: 'Homenaje a la neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Aunque Colvin es predominantemente conocida por temas abstractos que aluden a símbolos prehispánicos, esta gigantesca escultura, un homenaje a la neurocirugía, es una de sus obras de arte público más reconocibles.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Una estatua de bronce de dos manos cruzadas sosteniendo delicadamente un cerebro humano en la punta de sus dedos.'
}, {
  name: 'Floralis genérica',
  artist: 'Eduardo Catalano',
  description: 'Esta enorme flor plateada (75 pies o 23 m) se encuentra en Buenos Aires. Está diseñado para moverse, cerrando sus pétalos por la tarde o cuando soplan fuertes vientos y abriéndolos por la mañana.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Una gigantesca escultura de flor metálica con pétalos reflectantes como espejos y fuertes estambres.'
}, {
  name: 'Presencia eterna',
  artist: 'John Woodrow Wilson',
  description: 'Wilson era conocido por su preocupación por la igualdad, la justicia social, así como por las cualidades esenciales y espirituales de la humanidad. Este bronce masivo (7 pies o 2,13 m) representa lo que él describió como "una presencia negra simbólica infundida con un sentido de humanidad universal"."',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'La escultura que representa una cabeza humana parece omnipresente y solemne. Irradia calma y serenidad.'
}];
```

Muestra:

![[2-el-estado-1.png]]

**Es una buena idea tener múltiples variables de estado si no se encuentran relacionadas entre sí**, como `index` y `showMore` en este ejemplo. **Pero si encontramos que a menudo cambiamos dos variables de estado juntas, podría ser mejor combinarlas en una sola**. Por ejemplo, si tenemos un formulario con muchos campos, es más conveniente tener una única variable de estado que contenga un objeto que una variable de estado por campo. [Elegir la estructura del estado](https://es.react.dev/learn/choosing-the-state-structure) tiene más consejos sobre esto.

#### ¿Cómo sabe React qué estado devolver? 

Es posible que hayas notado que la llamada a `useState` no recibe ninguna información sobre _a qué_ variable de estado se refiere. No hay un «_identificador_» que se pase a `useState`, entonces, ¿cómo sabe cuál de las variables de estado debería devolver? ¿Se basa en algún tipo de magia para esto? La respuesta es no.

En cambio, para habilitar su sintaxis concisa, los Hooks **se basan en un orden de llamada estable en cada representación del mismo componente.** Esto funciona bien en la práctica porque si seguimos la regla anterior («_solo llamar a los Hooks en el nivel superior_»), los Hooks siempre se llamarán en el mismo orden. Además, este [complemento para el linter] ([https://www.npmjs.com/package/eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks)) detecta la mayoría de los errores.

Internamente, React mantiene un _array_ de pares de estados para cada componente. También mantiene el índice de par actual, el cual se establece en `0` antes de ser renderizado. Cada vez que llamamos a `useState`, React devuelve el siguiente par de estados e incrementa el índice. Puedes leer más sobre este mecanismo en [React Hooks: No es magia, sólo son Arrays.](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)

Este ejemplo **no usa React** pero nos da una idea de cómo funciona `useState` internamente:

**index.js**
```js
let componentHooks = [];
let currentHookIndex = 0;

// Cómo funciona useState dentro de React (simplificado).
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // Este no es el primer render,
    // entonces el par de estados ya existe.
    // Devuélvelo y prepárate para la próxima llamada del Hook.
    currentHookIndex++;
    return pair;
  }

  // Esta es la primera vez que estamos renderizando,
  // así que crea un array de dos posiciones y guárdalo.
  pair = [initialState, setState];

  function setState(nextState) {
    // Cuando el usuario solicita un cambio de estado,
    // guarda el nuevo valor en el par.
    pair[0] = nextState;
    updateDOM();
  }

  // Guarda el par para futuros renderizados
  // y se prepara para la siguiente llamada del Hook.
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}

function Gallery() {
  // Cada llamada a useState() devolverá el siguiente par.
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  // Este ejemplo no usa React, entonces
  // devuelve un objeto como resultado en lugar de JSX.
  return {
    onNextClick: handleNextClick,
    onMoreClick: handleMoreClick,
    header: `${sculpture.name} por ${sculpture.artist}`,
    counter: `${index + 1} of ${sculptureList.length}`,
    more: `${showMore ? 'Ocultar' : 'Mostrar'} detalles`,
    description: showMore ? sculpture.description : null,
    imageSrc: sculpture.url,
    imageAlt: sculpture.alt
  };
}

function updateDOM() {
  // Reinicia el índice del Hook actual
  // antes de renderizar el componente.
  currentHookIndex = 0;
  let output = Gallery();

  // Actualiza el DOM para que coincida con el resultado.
  // Esta es la parte que React hace por ti.
  nextButton.onclick = output.onNextClick;
  header.textContent = output.header;
  moreButton.onclick = output.onMoreClick;
  moreButton.textContent = output.more;
  image.src = output.imageSrc;
  image.alt = output.imageAlt;
  if (output.description !== null) {
    description.textContent = output.description;
    description.style.display = '';
  } else {
    description.style.display = 'none';
  }
}

let nextButton = document.getElementById('nextButton');
let header = document.getElementById('header');
let moreButton = document.getElementById('moreButton');
let description = document.getElementById('description');
let image = document.getElementById('image');
export const sculptureList = [{
  name: 'Homenaje a la neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Aunque Colvin es predominantemente conocida por temas abstractos que aluden a símbolos prehispánicos, esta gigantesca escultura, un homenaje a la neurocirugía, es una de sus obras de arte público más reconocibles.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Una estatua de bronce de dos manos cruzadas sosteniendo delicadamente un cerebro humano en la punta de sus dedos.'
}, {
  name: 'Floralis genérica',
  artist: 'Eduardo Catalano',
  description: 'Esta enorme flor plateada (75 pies o 23 m) se encuentra en Buenos Aires. Está diseñado para moverse, cerrando sus pétalos por la tarde o cuando soplan fuertes vientos y abriéndolos por la mañana.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Una gigantesca escultura de flor metálica con pétalos reflectantes como espejos y fuertes estambres.'
}, {
  name: 'Presencia eterna',
  artist: 'John Woodrow Wilson',
  description: 'Wilson era conocido por su preocupación por la igualdad, la justicia social, así como por las cualidades esenciales y espirituales de la humanidad. Este bronce masivo (7 pies o 2,13 m) representa lo que él describió como "una presencia negra simbólica infundida con un sentido de humanidad universal"."',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'La escultura que representa una cabeza humana parece omnipresente y solemne. Irradia calma y serenidad.'
}];

// Hacemos que la interfaz de usuario coincida con el estado inicial..
updateDOM();
```

**index.html**
```html
<button id="nextButton">
  Siguiente
</button>
<h3 id="header"></h3>
<button id="moreButton"></button>
<p id="description"></p>
<img id="image">

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
button { display: block; margin-bottom: 10px; }
</style>
```

Muestra:

![[2-el-estado-1.png]]

No es necesario que lo entiendas para usar React, pero podrías encontrarlo como un modelo mental útil.

## ⭐ El estado es aislado y privado.

**El estado es local para cada instancia de un componente** en la pantalla. En otras palabras, **si renderizas el mismo componente dos veces, ¡cada copia tendrá un estado completamente aislado!** Cambiar uno de ellos no afectará al otro.

En este ejemplo, el anterior componente de `Galería` se ha renderizado dos veces sin cambios en su lógica. Puedes intentar hacer clic en los botones dentro de cada una de las galerías. Observarás que su estado es independiente:

**App.jsx**
```jsx
import Gallery from './Gallery.js';

export default function Page() {
  return (
    <div className="Page">
      <Gallery />
      <Gallery />
    </div>
  );
}
```

**Gallery.jsx**
```jsx
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <section>
      <button onClick={handleNextClick}>
        Siguiente
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        por {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} de {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? 'Ocultar' : 'Mostrar'} detalles
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
    </section>
  );
}
```

**data.js**
```jsx
export const sculptureList = [{
  name: 'Homenaje a la neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Aunque Colvin es predominantemente conocida por temas abstractos que aluden a símbolos prehispánicos, esta gigantesca escultura, un homenaje a la neurocirugía, es una de sus obras de arte público más reconocibles.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Una estatua de bronce de dos manos cruzadas sosteniendo delicadamente un cerebro humano en la punta de sus dedos.'
}, {
  name: 'Floralis genérica',
  artist: 'Eduardo Catalano',
  description: 'Esta enorme flor plateada (75 pies o 23 m) se encuentra en Buenos Aires. Está diseñado para moverse, cerrando sus pétalos por la tarde o cuando soplan fuertes vientos y abriéndolos por la mañana.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Una gigantesca escultura de flor metálica con pétalos reflectantes como espejos y fuertes estambres.'
}, {
  name: 'Presencia eterna',
  artist: 'John Woodrow Wilson',
  description: 'Wilson era conocido por su preocupación por la igualdad, la justicia social, así como por las cualidades esenciales y espirituales de la humanidad. Este bronce masivo (7 pies o 2,13 m) representa lo que él describió como "una presencia negra simbólica infundida con un sentido de humanidad universal"."',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'La escultura que representa una cabeza humana parece omnipresente y solemne. Irradia calma y serenidad.'
}];
```

Muestra:

![[2-el-estado-2.png]]

**Esto es lo que hace que el estado sea diferente de las variables regulares que declaramos en la parte superior de un módulo**. El estado no está vinculado a una llamada de función en particular o a un lugar en el código, pero es «local» al lugar específico en la pantalla. **Se han renderizado dos componentes `<Gallery />`, por lo que su estado se almacena por separado**.

También observemos cómo el componente de la página `Page` no «sabe» nada sobre el estado del componente de la galería `Galery`, ni siquiera si es que posee algún estado. **A diferencia de las props, el estado es totalmente privado para el componente que lo declara.** **El componente padre no puede cambiarlo**. Esto permite agregar el estado a cualquier componente o eliminarlo sin afectar al resto de los componentes.

¿Qué pasaría si quisiéramos que ambas galerías mantuvieran sus estados sincronizados? La forma correcta de hacerlo en React es _eliminar_ el estado de los componentes secundarios y agregarlo a su padre más cercano. Las próximas páginas se centrarán en organizar el estado de un solo componente, pero volveremos a este tema en [Compartir estado entre componentes.](https://es.react.dev/learn/sharing-state-between-components)

## Recapitulación

- Debemos utilizar una variable de estado cuando necesitamos que un componente necesite «recordar» alguna información entre renderizaciones.
- Las variables de estado se declaran llamando al Hook `useState`.
- Los Hooks son funciones especiales que comienzan con `use`. Nos permiten «enlazarnos» a funciones de React como el estado.
- Evita llamar a Hooks de manera anidada (por ejemplo, dentro de bucles o condicionales). Llamar a Hooks -incluyendo al useState- solo es válido en el nivel superior de un componente u otro Hook.
- El Hook `useState` devuelve un _array_ de dos valores: el estado actual y la función para actualizarlo.
- Puede tener más de una variable de estado. Internamente, React los empareja por orden.
- El estado es privado para un componente. Si los renderizamos en dos lugares, cada componente lo maneja individualmente.