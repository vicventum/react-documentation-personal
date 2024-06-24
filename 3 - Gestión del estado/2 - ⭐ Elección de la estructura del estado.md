Estructurar bien el estado puede marcar la diferencia entre un componente que es agradable de modificar y depurar, y uno que es una fuente constante de errores. Estos son algunos consejos que debe tener en cuenta al estructurar el estado.

### Aprenderás

- Cuando usar una versus múltiples variables de estado.
- Qué evitar al organizar el estado.
- Cómo solucionar problemas comunes con la estructura del estado.

## ⭐Principios para la estructuración del estado

**Cuando escribes un componente que contiene algún estado, tendrás que tomar decisiones acerca de cuántas variables de estado usar y cuál debería ser la forma de tus datos**. Si bien es posible escribir programas correctos incluso con una estructura de estado deficiente, existen algunos principios que pueden guiarte para tomar mejores decisiones:

1. **Estado relacionado con el grupo.** Si siempre actualizas dos o más variables de estado al mismo tiempo, considera fusionarlas en una sola variable de estado.
2. **Evita las contradicciones en el estado.** Cuando el estado está estructurado de manera que varias partes del estado pueden contradecirse y «estar en desacuerdo» entre sí, deja espacio para errores. Trata de evitar esto.
3. **Evita el estado redundante.** Si puedes calcular alguna información de las propiedades del componente o sus variables de estado existentes durante el renderizado, no debes poner esa información en el estado de ese componente.
4. **Evita la duplicación de estado.** Cuando los mismos datos se duplican entre varias variables de estado o dentro de objetos anidados, es difícil mantenerlos sincronizados. Reduce la duplicación cuando puedas.
5. **Evita el estado profundamente anidado.** El estado profundamente jerárquico no es muy conveniente para actualizar. Cuando sea posible, prefiere estructurar el estado de forma plana.

**El objetivo detrás de estos principios es _hacer que el estado sea fácil de actualizar sin introducir errores_**. La eliminación de datos redundantes y duplicados del estado ayuda a garantizar que todas sus piezas permanezcan sincronizadas. Esto es similar a cómo un ingeniero de base de datos podría querer [«normalizar» la estructura de la base de datos](https://docs.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description) para reducir la posibilidad de errores. Parafraseando a Albert Einstein, **«_Haz que tu estado sea lo más simple posible, pero no más simple_».**

Ahora veamos cómo se aplican estos principios en acción.

## Estado relativo al grupo

En ocasiones, es posible que no estés seguro entre usar una o varias variables de estado.

¿Deberías hacer esto?

```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

¿O esto?

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Técnicamente, puedes usar cualquiera de estos enfoques. Pero **si algunas de las dos variables de estado siempre cambian juntas, podría ser una buena idea unificarlas en una sola variable de estado.** Entonces no olvidarás mantenerlos siempre sincronizados, como en este ejemplo donde al mover el cursor se actualizan ambas coordenadas del punto rojo:

**App.js**
```jsx
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  )
}
```

Muestra:

![[2-elección-de-la-estructura-del-estado-1.png]]

**Otro caso en el que agruparás datos en un objeto o una matriz es cuando no sabes cuántas partes diferentes del estado se necesitarán. Por ejemplo, es útil cuando tienes un formulario en el que el usuario puede agregar campos personalizados**.

>[!warning]
> **Si tu variable de estado es un objeto, recuerda que [no se puede actualizar solo un campo en él](https://es.react.dev/learn/updating-objects-in-state) sin copiar explícitamente los otros campos**. Por ejemplo, no puedes hacer `setPosition({ x: 100 })` pues en el ejemplo anterior no tendría la propiedad `y` en ningún lugar. En su lugar, si quisieras establecer solo la propiedad `x`, la definirías así `setPosition({ ...position, x: 100 })`, o las dividirías en dos variables de estado y harías `setX(100)`.

## ⭐ Evitar contradicciones en el estado - Usar una variable con múltiples valores

Aquí hay un formulario de comentarios de un hotel con variables de estado `isSending` y `isSent`:

**App.js**
```jsx
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false); // 👈
  const [isSent, setIsSent] = useState(false); // 👈

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true); // 👈
    await sendMessage(text);
    setIsSending(false); // 👈
    setIsSent(true); // 👈
  }

  if (isSent) {
    return <h1>¡Gracias por tu retroalimentación!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>¿Cómo fue tu estadía en El Poney Pisador?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Enviar
      </button>
      {isSending && <p>Enviando...</p>}
    </form>
  );
}

// Pretender enviar un mensaje.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

Muestra:

![[2-elección-de-la-estructura-del-estado-2.png]]

**Si bien este código funciona, deja la puerta abierta para estados «_imposibles_»**. Por ejemplo, si olvidas llamar a `setIsSent` y `setIsSending` juntos, puede terminar en una situación en la que tanto `isSending` como `isSent` son `true` al mismo tiempo. Cuanto más complejo sea tu componente, más difícil será entender lo que sucedió.

**Dado que `isSending` y `isSent` nunca deben ser `true` al mismo tiempo, es mejor reemplazarlos con una variable de estado `status` que puede tomar uno de _tres_ estados válidos:** `'typing'` (initial), `'sending'` y `'sent'`:

**App.js**
```jsx
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>¡Gracias por tu retroalimentación!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>¿Cómo fue tu estadía en El Poney Pisador?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Enviar
      </button>
      {isSending && <p>Enviando...</p>}
    </form>
  );
}

// Pretender enviar un mensaje.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

Todavía puedes declarar algunas constantes para mejorar la legibilidad:

```jsx
const isSending = status === 'sending';
const isSent = status === 'sent';
```

Pero no son variables de estado, por lo que no debes preocuparte de que no estén sincronizadas entre sí.

## ⭐ Evitar estado redundante

**Si puedes calcular alguna información de las props del componente o sus variables de estado existentes durante el renderizado, _no debes_ poner esa información en el estado de ese componente**.

Por ejemplo, toma este formulario. Funciona, pero ¿puedes encontrar algún estado redundante en él?

**App.js**
```jsx
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Vamos a registrarte</h2>
      <label>
        Nombre:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Apellido:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
       Su boleto será emitido a:<b>{fullName}</b>
      </p>
    </>
  );
}
```

Muestra:

![[2-elección-de-la-estructura-del-estado-3.png]]

aquí, `fullName` _no_ es una variable de estado. En cambio, se calcula durante el renderizado:

```jsx
const fullName = firstName + ' ' + lastName;
```

Como resultado, los controladores de cambios no necesitan hacer nada especial para actualizarlo. Cuando llamas a `setFirstName` o `setLastName`, activas una nueva representación y luego el siguiente `fullName` se calculará a partir de los nuevos datos.

#### ⭐ No reflejar props en el estado - Las props para inicializar estados no mutan el estado cuando ellas cambian

Un ejemplo común de estado redundante es un código como este:

```jsx
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
...
```

Aquí, una variable de estado `color` se inicializa en la prop `messageColor`. El problema es que **si el componente principal pasa un valor diferente de `messageColor` más adelante (por ejemplo, `'red'` en lugar de `'blue'`), ¡la _variable de estado_ `color`¡no se actualizará!** el estado solo se inicializa durante el primer renderizado.

**Esta es la razón por la que «_reflejar_» alguna prop en una variable de estado puede generar confusión. En su lugar, usa el accesorio `messageColor` directamente en tu código. Si deseas darle un nombre más corto, use una constante**:

```jsx
function Message({ messageColor }) {  
	const color = messageColor;
...
```

De esta forma, no se sincroniza con la prop que se pasa desde el componente principal.

**«_Reflejar_» props en estado solo tiene sentido cuando _quieres_ ignorar todas las actualizaciones de una prop en específico. Por convención, comienza el nombre de la prop con `initial` or `default` para aclarar que sus nuevos valores se ignoran**:

```jsx
function Message({ initialColor }) {
  // La variable de estado `color` contiene el *primer* valor de `initialColor`.
  // Se ignoran los cambios posteriores a la prop `initialColor`.
  const [color, setColor] = useState(initialColor);
...
```

## 🌟 Evita la duplicación en el estado

Este componente de lista de menú te permite elegir un solo refrigerio de viaje entre varios:

**App.js**
```jsx
import { useState } from 'react';

const initialItems = [ // 👈
  { title: 'pretzels', id: 0 },
  { title: 'crujiente de algas', id: 1 },
  { title: 'barra de granola', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0] // 👈
  );

  return (
    <>
      <h2>¿Cuál es tu merienda de viaje?</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.title}
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Seleccionar</button>
          </li>
        ))}
      </ul>
      <p>Seleccionaste {selectedItem.title}.</p>
    </>
  );
}
```

Muestra:

![[2-elección-de-la-estructura-del-estado-4.png]]

Actualmente, almacena el elemento seleccionado como un objeto en la variable de estado `selectedItem`. Sin embargo, esto no está bien: **el contenido de `selectedItem` es el mismo objeto que uno de los elementos dentro de la lista de `items`.** **Esto significa que la información sobre el elemento en sí está duplicada en dos lugares**.

¿Por qué es esto un problema? Hagamos que cada elemento sea editable:

**App.js**
```jsx
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crujiente de algas', id: 1 },
  { title: 'barra de granola', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState( // 👈
    items[0] // 👈
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>¿Cuál es tu merienda de viaje?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedItem(item); // 👈
            }}>Seleccionar</button>
          </li>
        ))}
      </ul>
      <p>Seleccionaste {selectedItem.title}.</p>
    </>
  );
}
```

Muestra: 

![[2-eleccion-de-la-estructura-del-estado-6.png]]

**Observe cómo si primero haces clic en «_Seleccionar_» en un elemento y _luego_ lo editas, la entrada se actualiza, pero la etiqueta en la parte inferior no refleja las ediciones**. **Esto se debe a que tienes un estado duplicado y te olvidaste de actualizar `selectedItem`**.

Aunque también podría actualizar `selectedItem`, una solución más fácil es eliminar la duplicación. En este ejemplo, **en lugar de un objeto `selectedItem`** (que crea una duplicación con objetos dentro de `items`), **mantienes `selectedId` en el estado, y _luego_ obtienes el `selectedItem` buscando en la matriz `items` un artículo con esa identificación**:

**App.js**
```jsx
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crujiente de algas', id: 1 },
  { title: 'barra de granola', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0); // 👈

  const selectedItem = items.find(item => // 👈
    item.id === selectedId
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>¿Cuál es tu merienda de viaje?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedId(item.id); // 👈
            }}>Seleccionar</button>
          </li>
        ))}
      </ul>
      <p>Seleccionaste {selectedItem.title}.</p>
    </>
  );
}
```

El estado solía duplicarse de esta manera:

- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedItem = {id: 0, title: 'pretzels'}`

Pero después del cambio es quedó de esta forma:

- `items = [{ id: 0, title: 'pretzels'}, ...]`
- `selectedId = 0`

¡La duplicación se ha ido, y solo conservas el estado esencial!

Ahora, si editas el ítem _seleccionado_, el siguiente mensaje se actualizará inmediatamente. Esto se debe a que `setItems` desencadena una nueva representación, y `items.find(...)` encontraría el ítem con el título actualizado. No era necesario mantener _el ítem seleccionado_ en el estado, porque solo el _ID seleccionado_ es esencial. El resto podría calcularse durante el renderizado.

## Evita el estado profundamente anidado

Imagina un plan de viaje compuesto por planetas, continentes y países. Es posible que sientas la tentación de estructurar tu estado mediante objetos y matrices anidados, como en este ejemplo:

**App.jsx**
```jsx
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ place }) {
  const childPlaces = place.childPlaces;
  return (
    <li>
      {place.title}
      {childPlaces.length > 0 && (
        <ol>
          {childPlaces.map(place => (
            <PlaceTree key={place.id} place={place} />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const planets = plan.childPlaces;
  return (
    <>
      <h2>Lugares para visitar</h2>
      <ol>
        {planets.map(place => (
          <PlaceTree key={place.id} place={place} />
        ))}
      </ol>
    </>
  );
}
```

**places.jsx**
```jsx
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Tierra',
    childPlaces: [{
      id: 2,
      title: 'África',
      childPlaces: [{
        id: 3,
        title: 'Botsuana',
        childPlaces: []
      }, {
        id: 4,
        title: 'Egipto',
        childPlaces: []
      }, {
        id: 5,
        title: 'Kenia',
        childPlaces: []
      }, {
        id: 6,
        title: 'Madagascar',
        childPlaces: []
      }, {
        id: 7,
        title: 'Marruecos',
        childPlaces: []
      }, {
        id: 8,
        title: 'Nigeria',
        childPlaces: []
      }, {
        id: 9,
        title: 'Sudáfrica',
        childPlaces: []
      }]
    }, {
      id: 10,
      title: 'Las Américas',
      childPlaces: [{
        id: 11,
        title: 'Argentina',
        childPlaces: []
      }, {
        id: 12,
        title: 'Brasil',
        childPlaces: []
      }, {
        id: 13,
        title: 'Barbados',
        childPlaces: []
      }, {
        id: 14,
        title: 'Canadá',
        childPlaces: []
      }, {
        id: 15,
        title: 'Jamaica',
        childPlaces: []
      }, {
        id: 16,
        title: 'México',
        childPlaces: []
      }, {
        id: 17,
        title: 'Trinidad y Tobago',
        childPlaces: []
      }, {
        id: 18,
        title: 'Venezuela',
        childPlaces: []
      }]
    }, {
      id: 19,
      title: 'Asia',
      childPlaces: [{
        id: 20,
        title: 'China',
        childPlaces: []
      }, {
        id: 21,
        title: 'India',
        childPlaces: []
      }, {
        id: 22,
        title: 'Singapur',
        childPlaces: []
      }, {
        id: 23,
        title: 'Corea del sur',
        childPlaces: []
      }, {
        id: 24,
        title: 'Tailandia',
        childPlaces: []
      }, {
        id: 25,
        title: 'Vietnam',
        childPlaces: []
      }]
    }, {
      id: 26,
      title: 'Europa',
      childPlaces: [{
        id: 27,
        title: 'Croacia',
        childPlaces: [],
      }, {
        id: 28,
        title: 'Francia',
        childPlaces: [],
      }, {
        id: 29,
        title: 'Alemania',
        childPlaces: [],
      }, {
        id: 30,
        title: 'Italia',
        childPlaces: [],
      }, {
        id: 31,
        title: 'Portugal',
        childPlaces: [],
      }, {
        id: 32,
        title: 'España',
        childPlaces: [],
      }, {
        id: 33,
        title: 'Turquía',
        childPlaces: [],
      }]
    }, {
      id: 34,
      title: 'Oceanía',
      childPlaces: [{
        id: 35,
        title: 'Australia',
        childPlaces: [],
      }, {
        id: 36,
        title: 'Bora Bora (Polinesia Francesa)',
        childPlaces: [],
      }, {
        id: 37,
        title: 'Isla de pascua (Chile)',
        childPlaces: [],
      }, {
        id: 38,
        title: 'Fiyi',
        childPlaces: [],
      }, {
        id: 39,
        title: 'Hawái (Estados Unidos)',
        childPlaces: [],
      }, {
        id: 40,
        title: 'Nueva Zelanda',
        childPlaces: [],
      }, {
        id: 41,
        title: 'Vanuatu',
        childPlaces: [],
      }]
    }]
  }, {
    id: 42,
    title: 'Luna',
    childPlaces: [{
      id: 43,
      title: 'Rheita',
      childPlaces: []
    }, {
      id: 44,
      title: 'Piccolomini',
      childPlaces: []
    }, {
      id: 45,
      title: 'Tycho',
      childPlaces: []
    }]
  }, {
    id: 46,
    title: 'Marte',
    childPlaces: [{
      id: 47,
      title: 'Corn Town',
      childPlaces: []
    }, {
      id: 48,
      title: 'Green Hill',
      childPlaces: []
    }]
  }]
};
```

Muestra:

![[2-eleccion-de-la-estructura-del-estado-7.png]]

Ahora, supongamos que deseas agregar un botón para eliminar un lugar que ya visitaste. ¿Cómo lo harías? [Actualizar el estado anidado](https://es.react.dev/learn/updating-objects-in-state#updating-a-nested-object) implica hacer copias de objetos desde la parte que cambió. **La eliminación de un lugar profundamente anidado implica copiar toda la cadena de lugares principal. Dicho código puede ser muy verboso**.

**Si el estado está demasiado anidado para actualizarse fácilmente, considera hacerlo «_plano_».** Esta es una manera de reestructurar estos datos. **En lugar de una estructura similar a un árbol donde cada `lugar` tiene una matriz de _sus lugares secundarios_, puedes hacer que cada lugar contenga una matriz de _sus ID de lugares secundarios_. Luego puedes almacenar un mapeo de cada ID de lugar al lugar correspondiente**.

Esta reestructuración de datos puede recordarte ver una tabla de base de datos:

**App.jsx**
```jsx
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ id, placesById }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      {childIds.length > 0 && (
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              placesById={placesById}
            />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Lugares a visitar</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            placesById={plan}
          />
        ))}
      </ol>
    </>
  );
}
```

**places.js**
```jsx
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Tierra',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'África',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  },
  3: {
    id: 3,
    title: 'Botsuana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egipto',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenia',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  },
  7: {
    id: 7,
    title: 'Marruecos',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'Sudáfrica',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Las Américas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brasil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  },
  14: {
    id: 14,
    title: 'Canadá',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'México',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad y Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25],   
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'India',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapur',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Corea del sur',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Tailandia',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vietnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Europa',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Croacia',
    childIds: []
  },
  28: {
    id: 28,
    title: 'Francia',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Alemania',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italia',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'España',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turquía',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Oceanía',
    childIds: [35, 36, 37, 38, 39, 40, 41],   
  },
  35: {
    id: 35,
    title: 'Australia',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (Polinesia Francesa)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Isla de Pascua (Chile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fiyi',
    childIds: []
  },
  39: {
    id: 40,
    title: 'Hawái (Estados Unidos)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Nueva Zelanda',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Luna',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tycho',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Marte',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

**Ahora que el estado es «plano» (también conocido como «_normalizado_»), la actualización de elementos anidados se vuelve más fácil.**

Para eliminar un lugar ahora, solo necesitas actualizar dos niveles de estado:

- La versión actualizada de su lugar _principal_ debería excluir el ID eliminado de su matriz `childIds`.
- La versión actualizada del objeto raíz de «_tabla_» debe incluir la versión actualizada del lugar principal.

Este es un ejemplo de cómo podrías hacerlo:

**App.jsx**
```jsx
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Crear una nueva versión del lugar principal
    // que no incluye ID del hijo.
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId)
    };
    // Actualizar el objeto de estado raíz...
    setPlan({
      ...plan,
      // ...para que tenga el padre este actualizado.
      [parentId]: nextParent
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Lugares a visitar</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Completado
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

**places.js**
```jsx
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Tierra',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'África',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  },
  3: {
    id: 3,
    title: 'Botsuana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egipto',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenia',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  },
  7: {
    id: 7,
    title: 'Marruecos',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'Sudáfrica',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Las Américas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brasil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  },
  14: {
    id: 14,
    title: 'Canadá',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'México',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad y Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25],   
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'India',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapur',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Corea del sur',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Tailandia',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vietnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Europa',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Croacia',
    childIds: []
  },
  28: {
    id: 28,
    title: 'Francia',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Alemania',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italia',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'España',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turquía',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Oceanía',
    childIds: [35, 36, 37, 38, 39, 40, 41],   
  },
  35: {
    id: 35,
    title: 'Australia',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (Polinesia Francesa)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Isla de Pascua (Chile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fiyi',
    childIds: []
  },
  39: {
    id: 40,
    title: 'Hawái (Estados Unidos)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Nueva Zelanda',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Luna',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tycho',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Marte',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

Muestra:

![[2-eleccion-de-la-estructura-del-estado-8.png]]

Puede anidar el estado tanto como desees, pero hacerlo «_plano_» puede resolver numerosos problemas. Facilita la actualización del estado y ayuda a garantizar que no haya duplicación en diferentes partes de un objeto anidado.

#### Mejorar el uso de memoria 

A veces, también puedes reducir el anidamiento de estados moviendo algunos de los estados anidados a los componentes secundarios. Esto funciona bien para el estado efímero de la interfaz de usuario que no necesita almacenarse, por ejemplo, si se pasa el cursor por encima de un elemento.

**App.jsx**
```jsx
import { useImmer } from 'use-immer';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, updatePlan] = useImmer(initialTravelPlan);

  function handleComplete(parentId, childId) {
    updatePlan(draft => {
      // Elimina los ID secundarios del lugar principal.
      const parent = draft[parentId];
      parent.childIds = parent.childIds
        .filter(id => id !== childId);

      // Olvida este lugar y todo su subárbol.
      deleteAllChildren(childId);
      function deleteAllChildren(id) {
        const place = draft[id];
        place.childIds.forEach(deleteAllChildren);
        delete draft[id];
      }
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Lugares a visitar</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Completado
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

**places.js**
```js
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Tierra',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'África',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  },
  3: {
    id: 3,
    title: 'Botsuana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egipto',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenia',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  },
  7: {
    id: 7,
    title: 'Marruecos',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'Sudáfrica',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Las Américas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brasil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  },
  14: {
    id: 14,
    title: 'Canadá',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'México',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad y Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25,],   
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'India',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapur',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Corea del sur',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Tailandia',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vietnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Europa',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Croacia',
    childIds: []
  },
  28: {
    id: 28,
    title: 'Francia',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Alemania',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italia',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'España',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turquía',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Oceanía',
    childIds: [35, 36, 37, 38, 39, 40, 41],   
  },
  35: {
    id: 35,
    title: 'Australia',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (Polinesia Francesa)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Isla de Pascua (Chile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fiyi',
    childIds: []
  },
  39: {
    id: 40,
    title: 'Hawái (Estados Unidos)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Nueva Zelanda',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Luna',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tycho',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Marte',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

## Recapitulación

- Si dos variables de estado siempre se actualizan juntas, considera combinarlas en una.
- Elige cuidadosamente tus variables de estado para evitar crear estados «imposibles».
- Estructura tu estado de una manera que reduzca las posibilidades de que cometas un error al actualizarlo.
- Evita el estado redundante y duplicado para que no necesites mantenerlo sincronizado.
- No pongas props _en_ estado a menos que desees evitar específicamente las actualizaciones.
- Para patrones de interfaz de usuario como la selección, mantén el ID o el índice en estado en lugar del objeto mismo.
- Si actualizar el estado profundamente anidado es complicado, intenta aplanarlo.

```ts
import { ref } from 'vue'

async function useFetch<T>(service: Function) {
  const data = ref<T>()
  const isError = ref(false)
  const isLoading = ref(false)
  const isFromRefetch = ref(false)

  async function fetchData() {
    console.log('🚀 ~ isFromRefetch:', isFromRefetch.value)
    isFromRefetch.value = false
    isLoading.value = true

    try {
      const resp = await service()
      data.value = resp
      isError.value = false
    } catch (error) {
      data.value = undefined
      isError.value = true
    }
    isLoading.value = false
  }

  function init() {
    console.log('🚀 ~ isFromRefetch:', isFromRefetch.value)
    isFromRefetch.value = false
    fetchData()
  }
  init()

  function refetch() {
    console.log('🚀 ~ refetch ~ isRefetch:', isFromRefetch.value)
    isFromRefetch.value = true
    fetchData()
  }

  return { data, isLoading, isError, refetch, isFromRefetch }
}

export { useFetch }
```