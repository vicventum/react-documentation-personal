A menudo querrás mostrar muchos componentes similares de una colección de datos. Puedes usar los [métodos de array de JavaScript](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array#) para manipular un array de datos. En esta página, usarás [`filter()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) y [`map()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/map) con React para filtrar y transformar tu array de datos en un array de componentes.

### Aprenderás

- Cómo renderizar componentes desde un array usando el método `map()` de JavaScript
- Cómo renderizar solo un componente específico usando `filter()` de JavaScript
- Cuándo y cómo usar las _keys_ de React

## Renderizar datos desde arrays

Digamos que tienes una lista de contenido.

```html
<ul>
  <li>Creola Katherine Johnson: matemática</li>
  <li>Mario José Molina-Pasquel Henríquez: químico</li>
  <li>Mohammad Abdus Salam: físico</li>
  <li>Percy Lavon Julian: químico</li>
  <li>Subrahmanyan Chandrasekhar: astrofísico</li>
</ul>
```

La única diferencia entre esos elementos de la lista es su contenido, sus datos. A menudo necesitarás mostrar muchas instancias del mismo componente usando diferentes datos cuando construyas interfaces: desde listas de comentarios a galerías de fotos de perfiles. En estas situaciones, puedes guardar estos datos en objetos de JavaScript y arrays, y usar métodos como [`map()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/map) y [`filter()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) para renderizar listas de componentes desde ellos.

Aquí hay un corto ejemplo de como generar una lista de elementos de un array:

1. **Mueve** los datos en un array:

```jsx
const people = [
  'Creola Katherine Johnson: matemática',
  'Mario José Molina-Pasquel Henríquez: químico',
  'Mohammad Abdus Salam: físico',
  'Percy Lavon Julian: químico',
  'Subrahmanyan Chandrasekhar: astrofísico'
];
```

2. **Mapea** los miembros de `people` en un nuevo array de nodos JSX, `listItems`:

```jsx
const listItems = people.map(person => <li>{person}</li>);
```

3. **Devuelve** `listItems` desde tu componente envuelto en un `<ul>`:

```jsx
return <ul>{listItems}</ul>;
```

Aquí está el resultado:

**App.js**
```jsx
const people = [
  'Creola Katherine Johnson: matemática',
  'Mario José Molina-Pasquel Henríquez: químico',
  'Mohammad Abdus Salam: físico',
  'Percy Lavon Julian: químico',
  'Subrahmanyan Chandrasekhar: astrofísico'
];

export default function List() {
  const listItems = people.map(person =>
    <li>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
```

Date cuenta que el sandbox anterior muestra un error por consola:

>[!danger]
>Warning: Each child in a list should have a unique «key» prop.
>**(Traducción)**
>Advertencia: Cada hijo en una lista debe tener una única prop «key».

Aprenderás como arreglar este error más adelante en esta página. Antes de que lleguemos a eso, vamos a añadir algo de estructura a tus datos.

## Filtrar arrays de objetos

Estos datos pueden ser incluso más estructurados.

```jsx
const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'matemática',
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'químico',
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'físico',
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'químico',  
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofísico',
}];
```

**Digamos que quieres una manera de mostrar solo las personas cuya profesión sea `'químico'`. Puedes usar el método `filter()` de JavaScript para devolver solo esas personas**. Este método coge un array de objetos, los pasa por un «_test_» (una función que devuelve `true` o `false`), y devuelve un nuevo array de solo esos objetos que han pasado el test (que han devuelto `true`).

Tú solo quieres los objetos donde `profession` es `'químico'`. La función «_test_» para esto se ve como `(person) => person.profession === 'químico'`. Aquí está cómo juntarlo:

1. **Crea** un nuevo array solo de personas que sean «químicos», `chemists`, llamando al método `filter()` en `people` filtrando por `person.profession === 'químico'`:

```jsx
const chemists = people.filter(person =>  person.profession === 'químico');
```

2. Ahora **mapea** sobre `chemists`:

```jsx
const listItems = chemists.map(person => // 👈
  <li>
     <img
       src={getImageUrl(person)}
       alt={person.name}
     />
     <p>
       <b>{person.name}:</b>
       {' ' + person.profession + ' '}
       conocido/a por {person.accomplishment}
     </p>
  </li>
);
```

3. Por último, **devuelve** el `listItems` de tu componente:

```jsx
return <ul>{listItems}</ul>;
```

**App.js**
```jsx
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'químico'
  );
  const listItems = chemists.map(person =>
    <li>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        conocido/a por {person.accomplishment}
      </p>
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

**utils.js**
```jsx
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

**data.js**
```jsx
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'matemática',
  accomplishment: 'los cálculos de vuelos espaciales',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'químico',
  accomplishment: 'el descubrimiento del agujero de ozono en el Ártico',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'físico',
  accomplishment: 'la teoría del electromagnetismo',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'químico',
  accomplishment: 'ser pionero en el uso de cortisona, esteroides y píldoras anticonceptivas',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofísico',
  accomplishment: 'los cálculos de masa de estrellas enanas blancas',
  imageId: 'lrWQx8l'
}];
```

Muestra:

![[7-renderizado-de-listas-1.png]]

> [!warning]
>Las funciones de flecha implícitamente devuelven la expresión justo después del `=>`, así que no necesitas declarar un `return`:
>
>```
>const listItems = chemists.map(person =>  <li>...</li> // Implicit return!);
>```
>
>Sin embargo, **¡debes escibir el `return` explícitamente si tu `=>` está seguida por una llave`{`!**
>
>```
>const listItems = chemists.map(person => { // Curly brace  return <li>...</li>;});
>```
>
>Las funciones de flecha que tienen `=> {` se dice que tienen un [«cuerpo de bloque».](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Functions/Arrow_functions#cuerpo_de_funci%C3%B3n) Te permiten escribir más de una sola línea de código, pero _tienes que_ declarar un `return` por ti mismo. Si lo olvidas, ¡Nada será devuelto!

## ⭐ Mantener los elementos de una lista en orden con _`key`_

Fíjate que todos los sandboxes anteriores mostraban un error en la consola:

>[!danger]
>Warning: Each child in a list should have a unique «key» prop.
>**(Traducción)**
>Advertencia: Cada hijo en una lista debe tener una única prop «key».

**Tienes que darle a cada elemento del array una _`key`_** (una cadena de texto o un número) **que lo identifique de manera única entre otros elementos del array**:

```jsx
<li key={person.id}>...</li>
```

> [!note]
>¡Los elementos JSX directamente dentro de una llamada a un `map()` siempre necesitan _keys_!

**Las _keys_ le indican a React que objeto del array corresponde a cada componente, para así poder emparejarlo más tarde**. **Esto se vuelve más importante si los objetos de tus arrays se pueden mover (p. ej. debido a un ordenamiento), insertar, o eliminar**. Una _`key`_ bien escogida ayuda a React a entender lo que ha sucedido exactamente, y hacer las correctas actualizaciones en el árbol del DOM.

En vez de generar _keys_ sobre la marcha, deberías incluirlas en tus datos:

**App.js**
```jsx
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}</b>
          {' ' + person.profession + ' '}
          conocido/a por {person.accomplishment}
      </p>
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

**utils.js**
```jsx
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

**data.js**
```jsx
export const people = [{
  id: 0, // Usado en JSX como key
  name: 'Creola Katherine Johnson',
  profession: 'matemática',
  accomplishment: 'los cálculos de vuelos espaciales',
  imageId: 'MK3eW3A'
}, {
  id: 1, // Usado en JSX como key
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'químico',
  accomplishment: 'el descubrimiento del agujero de ozono en el Ártico',
  imageId: 'mynHUSa'
}, {
  id: 2, // Usado en JSX como key
  name: 'Mohammad Abdus Salam',
  profession: 'físico',
  accomplishment: 'la teoría del electromagnetismo',
  imageId: 'bE7W1ji'
}, {
  id: 3, // Usado en JSX como key
  name: 'Percy Lavon Julian',
  profession: 'químico',
  accomplishment: 'ser pionero en el uso de cortisona, esteroides y píldoras anticonceptivas',
  imageId: 'IOjWm71'
}, {
  id: 4, // Usado en JSX como key
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofísico',
  accomplishment: 'los cálculos de masa de estrellas enanas blancas',
  imageId: 'lrWQx8l'
}];
```


#### ⭐ Mostrar varios nodos DOM para cada elemento de una lista

¿Qué haces cuándo cada objeto necesita renderizar no uno, sino varios nodos del DOM?

El atajo de sintaxis del [`<>...</>` Fragment](https://es.react.dev/reference/react/Fragment) no te dejará pasarle una key, así que **necesitas agruparlos en un solo `<div>`, o usar una sintaxis algo más larga y [más explícita del `<Fragment>**`:](https://es.react.dev/reference/react/Fragment#rendering-a-list-of-fragments)

```jsx
import { Fragment } from 'react';

// ...

const listItems = people.map(person =>
  <Fragment key={person.id}> // 👈
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
);
```

**Los Fragments desaparecen del DOM**, así que esto producirá una lista plana de `<h1>`, `<p>`, `<h1>`, `<p>`, y así.

### Dónde conseguir tu _`key`_

Distintas fuentes de datos dan diferentes fuentes de _keys_:

- **Datos de una base de datos:** Si tus datos vienen de una base de datos, puedes usar las _keys_/ID de la base de datos, que son únicas por naturaleza.
- **Datos generados localmente:** Si tus datos son generados y persistidos localmente (p. ej. notas en una app de tomar notas), usa un contador incremental, [`crypto.randomUUID()`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID) o un paquete como [`uuid`](https://www.npmjs.com/package/uuid) cuando este creando objetos.

### ⭐ Reglas de las _keys_

- **Las _keys_ tienen que ser únicas entre elementos hermanos.** Sin embargo, está bien usar las mismas _keys_ para nodos JSX en arrays _diferentes_.
- **Las _keys_ no tienen que cambiar** o ¡eso quitará su propósito! No las generes mientras renderizas.

### ⭐ ¿Por qué React necesita _keys_?

Imagina que los archivos de tu escritorio no tuvieran nombres. En vez de eso, tu te referirías a ellos por su orden — el primer archivo, el segundo, y así. Podrías acostumbrarte a ello, pero una vez borres un archivo, se volvería algo confuso. El segundo archivo se convertiría en el primero, el tercer archivo se convertiría en el segundo, y así.

Los nombres de archivos en una carpeta y las _keys_ JSX en un array tienen un propósito similar. Nos permiten identificar un objeto de manera única entre sus hermanos. Una _key_ bien escogida da más información aparte de la posición en el array. incluso si la _posición_ cambia devido a un reordenamiento, la _`key`_ permite a React identificar al elemento a lo largo de su ciclo de vida.

### ⭐ No usar el `index` de un bucle como `key`

> [!warning]
>**Podrías estar tentado a usar el índice del elemento en el array como su _key_. De hecho, eso es lo que React usará si tu no especifícas una _`key`_ en absoluto. Pero el orden en el que renderizas elementos cambiará con el tiempo si un elemento es insertado, borrado, o si se reordena su array**. El índice como _key_ lleva a menudo a sutiles y confusos errores.
>
>**Igualmente, no generes _keys_ sobre la marcha, p. ej. con `key={Math.random()}`**. **Esto hará que las _keys_ nunca coincidan entre renderizados, llevando a todos tus componentes y al DOM a recrearse cada vez**. No solo es una manera lenta, si no que también pierde cualquier input del usuario dentro de los elementos listados. En vez de eso, usa unas IDs basadas en datos.
>
>Date cuenta de que tus componentes no reciben la _`key`_ como un prop. Solo es usado como pista para React. Si tus componentes necesitan un ID, se lo tienes que pasar como una prop separada: `<Profile key={id} userId={id} />`.

## Recapitulación

En esta página has aprendido:

- Como mover datos fuera de componentes y en estructuras de datos como arrays y objetos.
- Como genrerar sets de componentes similares con el método `map()` de JavaScript.
- Como crear arrays de objetos filtrados con el método `filter()` de JavaScript.
- Por qué y cómo poner la _`key`_ en cada componente en una colección para que React pueda seguir la pista de cada uno de ellos incluso si su posición o datos cambia.