Los _arrays_ son mutables en JavaScript, pero deberían tratarse como inmutables cuando los almacenas en el estado. **Al igual que los objetos, cuando quieras actualizar un _array_ almacenado en el estado, necesitas crear uno nuevo (o hacer una copia de uno existente) y luego asignar el estado para que utilice este nuevo _array_**.

### Aprenderás

- Cómo añadir, eliminar o cambiar elementos en un _array_ en el estado de React
- Cómo actualizar un objeto dentro de un _array_
- Cómo copiar un _array_ de forma menos repetitiva con Immer

## 🌟 Actualizar _arrays_ sin mutación

En JavaScript, los _arrays_ son solo otro tipo de objeto. **[[6 - ⭐ Actualizar objetos en el estado|Tal como los objetos que vimos en la lección pasada]], deberías tratar los _arrays_ en el estado de React como si fueran de solo lectura.** Esto significa que no deberías reasignar elementos dentro de un _array_ como `arr[0] = 'pájaro'`, y tampoco deberías usar métodos que puedan mutar el _array_, como `push()` y `pop()`.

En su lugar, **cada vez que quieras actualizar un _array_, querrás pasar un _nuevo_ _array_ a la función de asignación de estado. Para hacerlo, puedes crear un nuevo _array_ a partir del _array_ original en el estado si llamas a sus métodos que no lo muten como `filter()` y `map()`. Luego puedes asignar el estado a partir del nuevo _array_ resultante**.

**Aquí hay una tabla de referencia con las operaciones más comunes con _arrays**_. Cuando se trata de _arrays_ dentro del estado de React, necesitarás evitar los métodos de la columna izquierda, y en su lugar es preferible usar los métodos de la columna derecha.

| Operaciones | evita (muta el _array_)         | preferido (devuelve un nuevo _array_)                                                                                                                                                                             |
| :---------: | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   obtener   |                                 | `at`, `find`, `findIndex`                                                                                                                                                                                         |
|   añadir    | `push`, `unshift`               | `concat` (agrega en la ultima posición), <br>`toSpliced` (agrega en cualquier posición)<br>`[...arr]` operador de propagación ([ejemplo](https://es.react.dev/learn/updating-arrays-in-state#adding-to-an-array)) |
|  eliminar   | `pop`, `shift`, `splice`        | `filter`, `toSpliced`, `slice` ([ejemplo](https://es.react.dev/learn/updating-arrays-in-state#removing-from-an-array)),                                                                                           |
| reemplazar  | `splice`, `arr[i] = ...` asigna | `toSpliced`, `width`, `map` ([ejemplo](https://es.react.dev/learn/updating-arrays-in-state#replacing-items-in-an-array))                                                                                          |
|   ordenar   | `reverse`, `sort`               | `toReversed`, `toSorted`, copia el _array_ primero ([ejemplo](https://es.react.dev/learn/updating-arrays-in-state#making-other-changes-to-an-array))                                                              |
|   others    |                                 | `some`, `every`, `fill`                                                                                                                                                                                           |

Como alternativa, puedes [usar Immer](https://es.react.dev/learn/updating-arrays-in-state#write-concise-update-logic-with-immer) el cual te permite usar métodos de ambas columnas.

>[!warning]
>Desafortunadamente, [`slice`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) y [`splice`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/splice) tienen nombres muy similares pero son muy diferentes:
>
>- `slice` te permite copiar un _array_ o una parte del mismo.
>- `splice` **muta** el _array_ (para insertar o eliminar elementos).
>
>En React, estarás usando `slice` (no `p`!) o `toSpliced` (que es directamente la versión no mutadora de `splice`) mucho más seguido porque no quieres mutar objetos o _arrays_ en el estado. [Actualizar objetos](https://es.react.dev/learn/updating-objects-in-state) explica qué es mutación y por qué no se recomienda para el estado.

### ⭐ Añadir a un _array_

`push()` muta un _array_, lo cual no queremos:

**App.js**
```jsx
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Escultores inspiradores:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        artists.push({
          id: nextId++,
          name: name,
        });
      }}>Añadir</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

Muestra:

![[7-actualizar-arrays-en-el-estado-1.png]]


En su lugar, crea un _nuevo_ _array_ que contenga los elementos existentes _y_ un nuevo elemento al final. Hay múltiples formas de hacerlo, pero la más fácil es usar la sintaxis `...` [de propagación en _arrays_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_array_literals):

```jsx
setArtists( // Reemplaza el estado
  [ // con el nuevo _array_
    ...artists, // el cual contiene todos los elementos antiguos
    { id: nextId++, name: name } // y un nuevo elemento al final
  ]
);
```

o usar el método `.concat()` que devuelve un nuevo array pero con un nuevo elemento agregado al final del array (sería com la versión _no mutante_ de `.push()`):

```jsx
setArtists( // Reemplaza el estado
  artists.concat({
	id: nextId++,
	name: name,
  });
);
```

Ahora funciona correctamente:

```jsx
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Escultores inspiradores:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        setArtists([ // 👈
          ...artists,
          { id: nextId++, name: name }
        ]);
      }}>Añadir</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

El operador de propagación también te permite anteponer un elemento al colocarlo _antes_ del original `...artists`:

```jsx
setArtists([
  { id: nextId++, name: name }, // 👈
  ...artists // Coloca los elementos antiguos al final
]);
```

De esta forma, el operador de propagación puede hacer el trabajo tanto de `push()` añadiendo al final del _array_ como de `unshift()` agregando al comienzo del _array_. ¡Pruébalo en el editor de arriba!

### ⭐ Eliminar elementos de un _array_

**La forma más fácil de eliminar un elemento de un _array_ es _filtrarlo_**. En otras palabras, producirás un nuevo _array_ que no contendrá ese elemento. Para hacerlo, usa el método `filter`, por ejemplo:

**App.js**
```jsx
import { useState } from 'react';

let initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [artists, setArtists] = useState(
    initialArtists
  );

  return (
    <>
      <h1>Escultores inspiradores:</h1>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>
            {artist.name}{' '}
            <button onClick={() => {
              setArtists(
                artists.filter(a =>
                  a.id !== artist.id
                )
              );
            }}>
              Eliminar
            </button>
          </li>
        ))}
      </ul>
    </>
  );
}
```

Muestra:

![[7-actualizar-arrays-en-el-estado-2.png]]

Haz click en el botón «Eliminar» varias veces, y mira su controlador de clics.

```jsx
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

Aquí, `artists.filter(a => a.id !== artist.id)` significa «crea un nuevo _array_ conformado por aquellos `artists` cuyos IDs son diferentes de `artist.id`». En otras palabras, el botón «_Eliminar_» de cada artista filtrará a _ese_ artista del _array_ y luego solicitará un rerenderizado con el _array_ resultante. Ten en cuenta que `filter` no modifica el _array_ original.

### ⭐ Transformar un _array_

**Si deseas cambiar algunos o todos los elementos del _array_, puedes usar `map()`** para crear un **nuevo** _array_. La función que pasarás a `map` puede decidir qué hacer con cada elemento, en función de sus datos o su índice (o ambos).

En este ejemplo, un _array_ contiene las coordenadas de dos círculos y un cuadrado. Cuando presionas el botón, mueve solo los círculos 50 píxeles hacia abajo. Lo hace produciendo un nuevo _array_ de datos usando `map()`:

**App.js**
```jsx
import { useState } from 'react';

let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];

export default function ShapeEditor() {
  const [shapes, setShapes] = useState(
    initialShapes
  );

  function handleClick() {
    const nextShapes = shapes.map(shape => {
      if (shape.type === 'square') {
        // No cambia
        return shape;
      } else {
        // Devuelve un nuevo círculo 50px abajo
        return {
          ...shape,
          y: shape.y + 50,
        };
      }
    });
    // Vuelve a renderizar con el nuevo _array_
    setShapes(nextShapes);
  }

  return (
    <>
      <button onClick={handleClick}>
        ¡Mueve los círculos hacia abajo!
      </button>
      {shapes.map(shape => (
        <div
          key={shape.id}
          style={{
          background: 'purple',
          position: 'absolute',
          left: shape.x,
          top: shape.y,
          borderRadius:
            shape.type === 'circle'
              ? '50%' : '',
          width: 20,
          height: 20,
        }} />
      ))}
    </>
  );
}
```

Muestra:

![[7-actualizar-arrays-en-el-estado-3.png]]

Al hacer click muestra:

![[7-actualizar-arrays-en-el-estado-4.png]]
### ⭐ Reemplazar elementos en un _array_ 

Es particularmente común querer reemplazar uno o más elementos en un _array_. Las asignaciones como `arr[0] = 'pájaro'` están mutando el _array_ original, por lo que para esto también querrás usar `map`.

**Para reemplazar un elemento, crea una un nuevo _array_ con `map`**. Dentro de la llamada a `map`, recibirás el índice del elemento como segundo argumento. Úsalo para decidir si devolver el elemento original (el primer argumento) o algo más:

**App.js**
```jsx
import { useState } from 'react';

let initialCounters = [
  0, 0, 0
];

export default function CounterList() {
  const [counters, setCounters] = useState(
    initialCounters
  );

  function handleIncrementClick(index) {
    const nextCounters = counters.map((c, i) => {
      if (i === index) {
        // Incrementa el contador de clics
        return c + 1;
      } else {
        // El resto no ha cambiado
        return c;
      }
    });
    setCounters(nextCounters);
  }

  return (
    <ul>
      {counters.map((counter, i) => (
        <li key={i}>
          {counter}
          <button onClick={() => {
            handleIncrementClick(i);
          }}>+1</button>
        </li>
      ))}
    </ul>
  );
}
```

Muestra:

![[7-actualizar-arrays-en-el-estado-5.png]]

Al hacer click:

![[7-actualizar-arrays-en-el-estado-6.png]]

También podríamos hacerlo de una manera más simple usando `width`, ed la siguiente forma:

**App.jsx**
```jsx
...
function handleIncrementClick(index) {
    const nextCounters = counters.with(index, counters[index] + 1); // 👈
    setCounters(nextCounters);
  }
...
```

### ⭐ Insertar en un _array_

A veces, es posible que desees insertar un elemento en una posición particular que no esté ni al principio ni al final. Para hacer esto, puedes usar la sintaxis de propagación para _arrays_ `...` junto con el método `slice()`. El método `slice()` te permite cortar una «_rebanada_» del _array_. Para insertar un elemento, crearás un _array_ que extienda el segmento _antes_ del punto de inserción, luego el nuevo elemento y luego el resto del _array_ original.

En este ejemplo, el botón «Insertar» siempre inserta en el índice `1`:

**App.js**
```jsx
import { useState } from 'react';

let nextId = 3;
const initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState(
    initialArtists
  );

  function handleClick() {
    const insertAt = 1; // Podría ser cualquier índice
    const nextArtists = [
      // Elementos antes del punto de inserción:
      ...artists.slice(0, insertAt),
      // Nuevo ítem:
      { id: nextId++, name: name },
      // Elementos después del punto de inserción:
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
    setName('');
  }

  return (
    <>
      <h1>Escultores inspiradores:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={handleClick}>
        Insertar
      </button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

Muestra:

![[7-actualizar-arrays-en-el-estado-7.png]]

Despues de insertar los valores `1`, y luego `2` muestra:

![[7-actualizar-arrays-en-el-estado-8.png]]

También una manera aún más sencilla de hacerlo es con el método `toSpliced`:

**App.jsx**
```jsx
...
function handleClick() {
    const insertAt = 1; // Podría ser cualquier índice
    const nextArtists = artists.toSpliced(insertAt, 0, { // 👈
	    id: nextId++, 
	    name: name 
	});
    
    setArtists(nextArtists);
    setName("");
  }
...
```

### Hacer otros cambios en un _array_

Hay algunas cosas que no puedes hacer con la sintaxis extendida y los métodos como `map()` y `filter()`. Por ejemplo, es posible que desees invertir u ordenar un _array_. Los métodos JavaScript `reverse()` y `sort()` mutan el _array_ original, por lo que no puedes usarlos directamente.

**Sin embargo, puedes copiar el _array_ primero y luego realizar cambios en él.**

**App.js**
```jsx
import { useState } from 'react';

const initialList = [
  { id: 0, title: 'Grandes barrigas' },
  { id: 1, title: 'Paisaje lunar' },
  { id: 2, title: 'Guerreros de terracota' },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
    const nextList = [...list]; // 👈
    nextList.reverse(); // 👈
    setList(nextList); // 👈
  }

  return (
    <>
      <button onClick={handleClick}>
        Invertir
      </button>
      <ul>
        {list.map(artwork => (
          <li key={artwork.id}>{artwork.title}</li>
        ))}
      </ul>
    </>
  );
}
```

Muestra:

![[7-actualizar-arrays-en-el-estado-9.png]]

Al hacer click en el botón, muestra:

![[7-actualizar-arrays-en-el-estado-10.png]]

Aquí, usas la sintaxis de propagación `[...list]` para crear primero una copia del _array_ original. Ahora que tienes una copia, puedes usar métodos de mutación como `nextList.reverse()` o `nextList.sort()`, o incluso asignar elementos individuales con `nextList[0] = "algo"`.


De igual manera una forma más sencilla es usar el método `toReversed`, que sería como la versión _no mutante_ de `reverse`, ya que no modifica el array original sino que devuelve uno nuevo con el orden de los valores invertidos:

**App.jsx**
```jsx
...
function handleClick() {
	setList(list.toReversed()); // 👈
}
...
```
#### ⭐ Tener cuidado con mutar valores de copias de arrays

Si bien con el primer ejemplo al crear una copia del array y manipular dicha copia nuestro código funciona, debemos tener cuidado con esto, ya que, **incluso si copias un _array_, no puedes mutar los elementos existentes _dentro_ de éste directamente.** Esto se debe a que la copia es superficial: el nuevo _array_ contendrá los mismos elementos que el original. Entonces, si modificas un objeto dentro del _array_ copiado, estás mutando el estado existente. Por ejemplo, un código como este es un problema.

```jsx
const nextList = [...list];
nextList[0].seen = true; // Problema: muta list[0]
setList(nextList);
```

Aunque `nextList` y `list` son dos _arrays_ diferentes, **`nextList[0]` y `list[0]` apuntan al mismo objeto.** Entonces, al cambiar `nextList[0].seen`, está también cambiando `list[0].seen`. ¡Esta es una mutación de estado que debes evitar! 

Puedes resolver este problema de forma similar a [actualizar objetos JavaScript anidados](https://es.react.dev/learn/updating-objects-in-state#updating-a-nested-object): copiando elementos individuales que deseas cambiar en lugar de mutarlos.

## ⭐ Actualizar objetos dentro de _arrays_ 

**Los objetos no están _realmente_ ubicados «_dentro_» de los _arrays_. Puede parecer que están «_dentro_» del código, pero cada objeto en un _array_ es un valor separado, al que «_apunta_» el _array_**. Es por eso que debe tener cuidado al cambiar campos anidados como `list[0]`. ¡La lista de obras de arte de otra persona puede apuntar al mismo elemento del _array_!

**Al actualizar el estado anidado, debe crear copias desde el punto en el que desea actualizar y hasta el nivel superior.** Veamos cómo funciona esto.

En este ejemplo, dos listas separadas de ilustraciones tienen el mismo estado inicial. Se supone que deben estar aislados, pero debido a una mutación, su estado se comparte accidentalmente y marcar una casilla en una lista afecta a la otra lista:

**App.js**
```jsx
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Grandes barrigas', seen: false },
  { id: 1, title: 'Paisaje lunar', seen: false },
  { id: 2, title: 'Guerreros de terracota', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setYourList(yourNextList);
  }

  return (
    <>
      <h1>Lista de deseos de arte</h1>
      <h2>Mi lista de obras de arte para ver:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Tu lista de obras de arte para ver:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

Muestra:

![[7-actualizar-arrays-en-el-estado-11.png]]

Al hacer click en el primer item de la primera lista, se marca también el primer item de la segunda lista:

![[7-actualizar-arrays-en-el-estado-12.png]]


El problema está en un código como este:

```jsx
const myNextList = [...myList];
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // 👎 Problema: muta un elemento existente
setMyList(myNextList);
```

**Aunque el _array_ `myNextList` en sí mismo es nuevo, los _propios elementos_ son los mismos que en el _array_ `myList` original. Entonces, cambiar `artwork.seen` cambia el elemento de la obra de arte _original_**. Ese elemento de la obra de arte también está en `yourArtworks`, lo que causa el error. Puede ser difícil pensar en errores como este, pero afortunadamente desaparecen si evitas mutar el estado.

**Puedes usar `map` para sustituir un elemento antiguo con su versión actualizada sin mutación.**

```jsx
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // Crea un *nuevo* objeto con cambios // 👍
    return { ...artwork, seen: nextSeen };
  } else {
     // No cambia
    return artwork;
  }
}));
```

Aquí, `...` es la sintaxis de propagación de objetos utilizada para [crear una copia de un objeto.](https://es.react.dev/learn/updating-objects-in-state#copying-objects-with-the-spread-syntax)

Con este enfoque, ninguno de los elementos del estado existentes se modifica y el error se soluciona:

**App.js**
```jsx
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Grandes barrigas', seen: false },
  { id: 1, title: 'Paisaje lunar', seen: false },
  { id: 2, title: 'Guerreros de terracota', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    setMyList(myList.map(artwork => { // 👈
      if (artwork.id === artworkId) {
        // Crea un *nuevo* objeto con cambios
        return { ...artwork, seen: nextSeen };
      } else {
        // No cambia
        return artwork;
      }
    }));
  }

  function handleToggleYourList(artworkId, nextSeen) {
    setYourList(yourList.map(artwork => {
      if (artwork.id === artworkId) {
        // Crea un *nuevo* objeto con cambios
        return { ...artwork, seen: nextSeen };
      } else {
        // No cambia
        return artwork;
      }
    }));
  }

  return (
    <>
      <h1>Lista de deseos de arte</h1>
      <h2>Mi lista de obras de arte para ver:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Tu lista de obras de arte para ver:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```


Muestra:

![[7-actualizar-arrays-en-el-estado-11.png]]

Ahora al hacer click en el primer item de la primera lista, sólo se marcará ese item:

![[7-actualizar-arrays-en-el-estado-13.png]]

En general, **solo debes mutar objetos que acabas de crear.** Si estuvieras insertando una _nueva_ obra de arte, podrías mutarla, pero si se trata de algo que ya está en el estado, debes hacer una copia.

### ⭐ Escribe una lógica de actualización concisa con Immer

Actualizar _arrays_ anidados sin mutación puede volverse un poco repetitivo. [Al igual que con los objetos](https://es.react.dev/learn/updating-objects-in-state#write-concise-update-logic-with-immer):

- En general, no deberías necesitar actualizar el estado más de un par de niveles de profundidad. Si tus objetos de estado son muy profundos, es posible que desees [reestructurarlos de manera diferente](https://es.react.dev/learn/choosing-the-state-structure#avoid-deeply-nested-state) para que sean planos.
- Si no deseas cambiar la estructura de tu estado, puedes preferir usar [Immer](https://github.com/immerjs/use-immer), que te permite escribir usando la sintaxis conveniente, pero que realiza mutaciones, y se encarga de producir las copias por ti.

Aquí está el anterior ejemplo de una Lista de deseos de arte reescrito con Immer:

**App.jsx**
```jsx
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Grandes barrigas', seen: false },
  { id: 1, title: 'Paisaje lunar', seen: false },
  { id: 2, title: 'Guerreros de terracota', seen: true },
];

export default function BucketList() {
  const [myList, updateMyList] = useImmer( // 👈
    initialList
  );
  const [yourList, updateYourList] = useImmer( // 👈
    initialList
  );

  function handleToggleMyList(id, nextSeen) {
    updateMyList(draft => { // 👈
      const artwork = draft.find(a =>
        a.id === id
      );
      artwork.seen = nextSeen;
    });
  }

  function handleToggleYourList(artworkId, nextSeen) {
    updateYourList(draft => { // 👈
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Lista de deseos de arte</h1>
      <h2>Mi lista de obras de arte para ver:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Tu lista de obras de arte para ver:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

Ten en cuenta cómo con Immer, **la mutación como `artwork.seen = nextSeen` ahora está bien:**

```jsx
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

Esto se debe a que no está mutando el estado _original_, sino que está mutando un objeto `draft` especial proporcionado por Immer. Del mismo modo, puedes aplicar métodos de mutación como `push()` y `pop()` al contenido del `draft`.

Tras bambalinas, Immer siempre construye el siguiente estado desde cero de acuerdo con los cambios que ha realizado en el `draft`. Esto mantiene tus controladores de eventos muy concisos sin mutar nunca el estado.

## Recapitulación

- Puedes poner _arrays_ en el estado, pero no puedes cambiarlos.
- En lugar de mutar un _array_, crea una _nueva_ versión y actualiza el estado.
- Puedes usar la sintaxis de propagación `[...arr, newItem]` para crear _arrays_ con nuevos elementos.
- Puedes usar `filter()` y `map()` para crear nuevos _arrays_ con elementos filtrados o transformados.
- Puedes usar Immer para mantener tu código conciso.