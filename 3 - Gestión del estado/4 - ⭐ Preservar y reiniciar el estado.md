El estado está aislado entre los componentes. **React mantiene un registro de qué estado pertenece a qué componente basándose en su lugar en el árbol de la interfaz de usuario (_UI_)**. Puedes controlar cuándo preservar el estado y cuándo reiniciarlo entre rerenderizados.

### Aprenderás

- Cuando React elige preservar o restablecer el estado
- Cómo forzar a React a restablecer el estado del componente
- Cómo las _keys_ y los tipos afectan la preservación del estado.

## ⭐ El estado está atado a la posición en el árbol de renderizado

React construye [árboles de renderizado](https://es.react.dev/learn/understanding-your-ui-as-a-tree#the-render-tree) para la estructura de componentes en tu UI.

Cuando le das estado a tu componente, podrías pensar que el estado «vive» dentro del componente. Pero el estado en realidad se guarda dentro de React. **React asocia cada pieza de estado que mantiene con el componente correcto por la posición en la que se encuentra ese componente en el árbol de renderizado**.

En este caso, sólo hay una etiqueta JSX `<Counter />`, pero se representa en dos posiciones diferentes:

**App.js**
```jsx
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}
```

Muestra:

![[4-perservar-y-reiniciar-el-estado-1.png]]

Al pulsar un botón, sólo se actualiza un componente, pese a que se están instanciando 2 veces el mismo componente:

![[4-perservar-y-reiniciar-el-estado-2.png]]

Esta sería la apariencia del árbol:

![[4-perservar-y-reiniciar-el-estado-3.webp|600]]
>Árbol de React

**Son dos contadores separados porque cada uno se renderiza en su propia posición en el árbol.** Normalmente no tienes que pensar en estas posiciones para usar React, pero puede ser útil para entender cómo funciona.

**En React, cada componente en la pantalla tiene un estado totalmente aislado**. Por ejemplo, si renderizas dos componentes `Counter`, uno al lado del otro, cada uno de ellos obtendrá sus propios e independientes estados `score` y `hover`.

Prueba a hacer clic en ambos contadores y observa que no se afectan mutuamente:

**App.jsx**
```jsx
import { useState } from 'react';

export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}
```
Muestra:

![[4-perservar-y-reiniciar-el-estado-1.png]]

Al pulsar un botón, sólo se actualiza un componente, pese a que se están instanciando 2 veces el mismo componente:

![[4-perservar-y-reiniciar-el-estado-2.png]]

Como puedes ver, cuando se actualiza un contador, sólo se actualiza el estado de ese componente:

![[4-perservar-y-reiniciar-el-estado-4.webp|600]]
> Actualización del estado

**React mantendrá el estado mientras se renderice el mismo componente en la misma posición en el árbol**. Para ver esto, incrementa ambos contadores, luego quita el segundo componente desmarcando la casilla «Renderizar el segundo contador», y luego vuelve a añadirlo marcándola de nuevo:

**App.js**
```jsx
import { useState } from 'react';

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />} 
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={e => {
            setShowB(e.target.checked)
          }}
        />
        Renderizar el segundo contador
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}
```

Muestra:

![[4-perservar-y-reiniciar-el-estado-5.png]]

Al agregar uno en el segundo contador:

![[4-perservar-y-reiniciar-el-estado-6.png]]

Al borrar deshabilitar el segundo contador pulsando en la casilla:

![[4-perservar-y-reiniciar-el-estado-7.png]]

Al volver a habilitar el segundo contador vemos como el estado se ha reiniciado:

![[4-perservar-y-reiniciar-el-estado-8.png]]

**Observa cómo en el momento en que dejas de renderizar el segundo contador, su estado desaparece por completo. Eso es porque cuando React elimina un componente, destruye su estado.**

![[4-perservar-y-reiniciar-el-estado-9.webp|500]]
>Eliminación de un componente

Al marcar «Renderizar el segundo contador», se inicializa un segundo `Counter` y su estado se inicializa desde cero (`score = 0`) y se añade al DOM.

![[4-perservar-y-reiniciar-el-estado-10.webp|500]]
>Añadiendo un componente

**React preserva el estado de un componente mientras se renderiza en su posición en el árbol de la interfaz de usuario. Si se elimina, o se renderiza un componente diferente en la misma posición, React descarta su estado**.

## ⭐ El mismo componente en la misma posición preserva el estado 

En este ejemplo, hay dos tipos diferentes de etiquetas `<Counter />`:

**App.js**
```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> // 👈
      ) : (
        <Counter isFancy={false} />  // 👈
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Usar un estilo elegante
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}

```
Muestra:

![[4-perservar-y-reiniciar-el-estado-11.png]]

Al agregar uno:

![[4-perservar-y-reiniciar-el-estado-12.png]]

Al agregar otro pero esta vez marcando la casilla:

![[4-perservar-y-reiniciar-el-estado-13.png]]

**Cuando se marca o desactiva la casilla, el estado del contador no se reinicia**. Tanto si `isFancy` es `true` como si es `false`, siempre tendrás un `<Counter />` como primer hijo del `div` devuelto desde el componente raíz `App`:

![[4-perservar-y-reiniciar-el-estado-14.webp|550]]

**La actualización del estado de la `App` no reinicia el `Counter` porque el `Counter` permanece en la misma posición**.

**Es el mismo componente en la misma posición, por lo tanto desde la perspectiva de React, es el mismo contador**.


#### A React le importa es la posición del componente en el arbol de componentes, no el marcado HTML/JSX

¡Recuerda que **es la posición en el árbol de la UI —no en el markup JSX— lo que le importa a React!** Este componente tiene dos cláusulas `return` con diferentes etiquetas JSX `<Counter />` dentro y fuera del `if`:

**App.jsx**
```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return ( // 👈
      <div>
        <Counter isFancy={true} /> // 👈
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={e => {
              setIsFancy(e.target.checked)
            }}
          />
          Usar un estilo elegante
        </label>
      </div>
    );
  }
  return ( // 👈
    <div>
      <Counter isFancy={false} /> // 👈
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Usar un estilo elegante
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}
```
Se podría esperar que el estado se reiniciara al marcar la casilla de verificación, pero no es así. Esto se debe a que **las dos etiquetas `<Counter />` se renderizan en la misma posición.** React no sabe dónde colocas las condiciones en tu función. Todo lo que «ve» es el árbol que devuelves. En ambos casos, el componente `App` devuelve un `<div>` con `<Counter />` como primer hijo. Por eso React los considera como _el mismo_ `<Counter />`.

Puedes pensar que tienen la misma «dirección»: el primer hijo del primer hijo de la raíz. Así es como React los hace coincidir entre los renderizados anteriores y los siguientes, independientemente de cómo estructures tu lógica.

## ⭐ Diferentes componentes en la misma posición reinician el estado

En este ejemplo, al marcar la casilla de verificación se sustituirá `<Counter>` por un `<p>`:

**App.js**
```jsx
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? ( // 👈
        <p>¡Nos vemos luego!</p> // 👈
      ) : (
        <Counter />  // 👈
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Tómate un descanso
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}
```

Muestra:

![[4-perservar-y-reiniciar-el-estado-14.png]]

Al pulsar el boton muestra:

![[4-perservar-y-reiniciar-el-estado-15.png]]

Al pulsar la casilla muestra:

![[4-perservar-y-reiniciar-el-estado-16.png]]

Al volver a pulsar en la casilla vemos como el estado se ha reiniciado:

![[4-perservar-y-reiniciar-el-estado-14.png]]

Aquí se cambia entre _diferentes_ tipos de componentes en la misma posición. Inicialmente, el primer hijo del `<div>` contenía un `Counter`. Pero cuando lo cambiaste por un `p`, React eliminó el `Counter` del árbol de la UI y destruyó su estado.

![[4-perservar-y-reiniciar-el-estado-17.webp|660]]

Cuando `Counter` cambia a `p`, se borra el `Counter` y se añade `p`

![[4-perservar-y-reiniciar-el-estado-18.webp]]

Al volver a cambiar, se borra `p` y se añade el `Counter`.

Además, **cuando se renderiza un componente diferente en la misma posición, se reinicia el estado de todo su subárbol.** Para ver cómo funciona, incrementa el contador y luego marca la casilla:

**App.js**
```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div> // 👈
          <Counter isFancy={true} />  // 👈
        </div>
      ) : (
        <section> // 👈
          <Counter isFancy={false} /> // 👈
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Usar un estilo elegante
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}
```

Muestra: 

![[4-perservar-y-reiniciar-el-estado-18.png]]

Agregando uno al pulsar el botón:

![[4-perservar-y-reiniciar-el-estado-19.png]]

Pulsando en la casilla, vemos como se reinicia el estado:

![[4-perservar-y-reiniciar-el-estado-20.png]]

El estado del contador se reinicia cuando se hace clic en la casilla de verificación. **Aunque se renderiza un `Counter`, el primer hijo del `div` cambia de `div` a `section`. Cuando el `div` hijo se eliminó del DOM, todo el árbol debajo de él (incluyendo el `Counter` y su estado) se destruyó también**.

![[4-perservar-y-reiniciar-el-estado-21.webp]]

Cuando `section` cambia a `div`, se elimina la `section` y se añade el nuevo `div`

![[4-perservar-y-reiniciar-el-estado-22.webp]]

Al volver a cambiar, se elimina el `div` y se añade la nueva `section`.

Como regla general, **si quieres preservar el estado entre rerenderizados, la estructura de tu árbol necesita «coincidir» de un render a otro. Si la estructura es diferente, el estado se destruye porque React destruye el estado cuando elimina un componente del árbol**.

#### Por ello no se deben definir componentes anidados

Es por este motivo que no se deben anidar las definiciones de las funciones de los componentes.

Aquí, la función del componente `MyTextField` se define _dentro_ de `MyComponent`:

**App.js**
```jsx
import { useState } from 'react';

export default function MyComponent() { // 👈
  const [counter, setCounter] = useState(0);

  function MyTextField() { // 👈
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>Hiciste clic {counter} veces</button>
    </>
  );
}
```

Muestra:

![[4-perservar-y-reiniciar-el-estado-22.png]]

Escribiendo en el input:

![[4-perservar-y-reiniciar-el-estado-23.png]]

Haciendo clic en el botón, vemos cómo desaparece el input:

![[4-perservar-y-reiniciar-el-estado-24.png]]

**Cada vez que se hace clic en el botón, el estado de la entrada desaparece. Esto se debe a que se crea una función _diferente_ de `MyTextField` para cada renderizado de `MyComponent`. Estás renderizando un componente _diferente_ en la misma posición, por lo que React reinicia todo el estado que esté anidado por debajo**. Esto conlleva a errores y problemas de rendimiento. Para evitar este problema, **declara siempre las funciones del componente en el nivel superior, y no anides sus definiciones.**

## ⭐ Reiniciar el estado en la misma posición

**Por defecto, React preserva el estado de un componente mientras permanece en la misma posición**. Normalmente, esto es exactamente lo que quieres, así que tiene sentido como comportamiento por defecto. Pero a veces, **es posible que quieras reiniciar el estado de un componente**. Considera esta aplicación que permite a dos jugadores llevar la cuenta de sus puntuaciones durante cada turno:

**App.js**
```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        ¡Siguiente jugador!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>Puntos de {person}: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}
```

Muestra:

![[4-perservar-y-reiniciar-el-estado-25.png]]

Agrega uno al jugador Taylor:

![[4-perservar-y-reiniciar-el-estado-26.png]]

Damos click al botón para ir al siguiente jugador, vemos cómo los puntos del anterior jugador se mantienen:

![[4-perservar-y-reiniciar-el-estado-27.png]]

Actualmente, cuando se cambia de jugador, la puntuación se conserva. **Los dos `Counter` aparecen en la misma posición, por lo que React los ve como _el mismo_ `Counter` cuya prop `person` ha cambiado**.

Pero conceptualmente, en esta aplicación deberían ser dos contadores separados. Podrían aparecer en el mismo lugar en la UI, pero uno es un contador para Taylor, y otro es un contador para Sarah.

Hay dos maneras de reiniciar el estado al cambiar entre ellos:

1. Renderizar los componentes en diferentes posiciones
2. Dar a cada componente una identidad explícita con `key`.

### Opción 1: Renderizar un componente en diferentes posiciones

Si quieres que estos dos `Counter` sean independientes, **puedes representarlos en dos posiciones diferentes**:

**App.js**
```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA && // 👈
        <Counter person="Taylor" /> // 👈
      }
      {!isPlayerA && // 👈
        <Counter person="Sarah" /> // 👈
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        ¡Siguiente jugador!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>Puntos de {person}: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}
```

- Inicialmente, `isPlayerA` es `true`. Así que la primera posición contiene el estado `Counter`, y la segunda está vacía.
- Cuando haces clic en el botón «Siguiente jugador», la primera posición se borra, pero la segunda contiene ahora un ‘Counter’.

![[4-perservar-y-reiniciar-el-estado-28.png|680]]


Muestra:

![[4-perservar-y-reiniciar-el-estado-25.png]]

Agrega uno al jugador Taylor:

![[4-perservar-y-reiniciar-el-estado-26.png]]

Ahora al ir al siguiente jugador, vemos que el estado del contador se reinicia, y no concerva el punto del anterior jugador:

![[4-perservar-y-reiniciar-el-estado-29.png]]

El estado de cada `Counter` se destruye cada vez que se elimina del DOM. Por eso se reinician cada vez que se hace clic en el botón.

**Esta solución es conveniente cuando sólo tienes unos pocos componentes independientes renderizados en el mismo lugar. En este ejemplo, sólo tienes dos, por lo que no es una molestia renderizar ambos por separado en el JSX**.

### 🌟 Option 2: Opción 2: Reiniciar el estado con una _key_

También hay otra forma, más genérica, de reiniciar el estado de un componente.

Es posible que hayas visto _`key`_ al [renderizar listas.](https://es.react.dev/learn/rendering-lists#keeping-list-items-in-order-with-key) **Las _keys_ no son sólo para las listas. Puedes usar _keys_ para que React distinga entre cualquier componente**. Por defecto, React utiliza el orden dentro del padre («primer contador», «segundo contador») para discernir entre los componentes. Pero **las _keys_ te permiten decirle a React que no es sólo un _primer_ contador, o un _segundo_ contador, sino un contador específico**; por ejemplo, el contador de _Taylor_. De esta manera, React conocerá el contador de _Taylor_ dondequiera que aparezca en el árbol!

En este ejemplo, los dos `<Counter />` no comparten estado aunque aparezcan en el mismo lugar en JSX:

**App.js**
```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" /> // 👈
      ) : (
        <Counter key="Sarah" person="Sarah" /> // 👈
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        ¡Siguiente jugador!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>Puntos de {person}: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Agregar uno
      </button>
    </div>
  );
}

```

El cambio entre Taylor y Sarah no preserva el estado. Esto se debe a que **le asignaste diferentes _`keys`_:**

```jsx
{isPlayerA ? (
  <Counter key="Taylor" person="Taylor" />
) : (
  <Counter key="Sarah" person="Sarah" />
)}
```

**Especificar una _`key`_ le dice a React que use la propia _`key`_ como parte de la posición, en lugar de su orden dentro del padre. Por eso, aunque los renderices en el mismo lugar en JSX, desde la perspectiva de React, son dos contadores diferentes. Como resultado, nunca compartirán estado**. Cada vez que un contador aparece en la pantalla, su estado se crea. Cada vez que se elimina, su estado se destruye. Alternar entre ellos reinicia su estado una y otra vez.

> [!note]
> Recuerda que las _keys_ no son únicas globalmente. Sólo especifican la posición _dentro del padre_.

### ⭐ Reiniciar un formulario con una _key_

Reiniciar el estado con una _key_ es especialmente útil cuando se trata de formularios.

En esta aplicación de chat, el componente `<Chat>` contiene el estado del cuadro de texto:

**App.js**
```jsx
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

**ContactList.js**
```jsx
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}> // 👈
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

**Chat.js**
```jsx
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Chatear con ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Enviar a {contact.email}</button>
    </section>
  );
}
```

Muestra:

![[4-perservar-y-reiniciar-el-estado-30.png]]

Escribimos algo en el input:

![[4-perservar-y-reiniciar-el-estado-31.png]]

Al cambiar a otro contácto, vemos cómo el texto permanece, cosa que no queremos:

![[4-perservar-y-reiniciar-el-estado-32.png]]

Prueba a introducir algo en el cuadro de texto y luego pulsa «Alice» o «Bob» para elegir un destinatario diferente. Notarás que **el estado del cuadro de texto se conserva porque el `<Chat>` se renderiza en la misma posición en el árbol.**

**En muchas aplicaciones, este puede ser el comportamiento deseado, pero no en una aplicación de chat!**. No quieres que el usuario envíe un mensaje que ya ha escrito a una persona equivocada debido a un clic accidental. Para solucionarlo, añade una `key`:

```jsx
<Chat key={to.id} contact={to} />
```

**Esto asegura que cuando selecciones un destinatario diferente, el componente `Chat` se recreará desde cero**, incluyendo cualquier estado en el árbol que esté por debajo. React también recreará los elementos del DOM en lugar de reutilizarlos.

Ahora al cambiar de destinatario siempre se borra el campo de texto:

**App.jsx**
```jsx
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.id} contact={to} />  // 👈
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```
#### Preservar el estado de los componentes removidos

En una aplicación de chat real, probablemente querrás recuperar el estado de la entrada cuando el usuario vuelva a seleccionar el destinatario anterior. Hay algunas maneras de mantener el estado «vivo» para un componente que ya no es visible:

- Podrías mostrar _todos_ los chats en lugar de sólo el actual, pero ocultar todos los demás con CSS. Los chats no se eliminarían del árbol, por lo que su estado local se conservaría. Esta solución funciona muy bien para UIs simples. Pero puede ser muy lenta si los árboles ocultos son grandes y contienen muchos nodos DOM.
- Podrías [subir el estado](https://es.react.dev/learn/sharing-state-between-components) y mantener el mensaje pendiente para cada destinatario en el componente padre. De esta manera, cuando los componentes hijos se eliminan, no importa, porque es el padre el que mantiene la información importante. Esta es la solución más común. También podrías utilizar una fuente diferente además del estado de React. Por ejemplo, probablemente quieras que el borrador de un mensaje persista incluso si el usuario cierra accidentalmente la página. Para implementar esto, podrías hacer que el componente `Chat` inicialice su estado leyendo de [`localStorage`](https://developer.mozilla.org/es/docs/Web/API/Window/localStorage) y guardar los borradores allí también.

Independientemente de la estrategia que elijas, un chat _con Alice_ es conceptualmente distinto de un chat _con Bob_, por lo que tiene sentido dar una _`key`_ al árbol `<Chat>` basado en el destinatario actual.

## ⭐ Recapitulación

- React mantiene el estado mientras el mismo componente se renderice en la misma posición.
- El estado no se mantiene en las etiquetas JSX. Se asocia a la posición del árbol en la que se coloca ese JSX.
- Puedes forzar a un subárbol a reiniciar su estado dándole una _key_ diferente.
- No anides las definiciones de los componentes, o reiniciarás el estado por accidente.