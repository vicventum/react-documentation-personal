## Desafío 1 de 4: Transformar datos sin Efectos

El `TodoList` a continuación muestra una lista de tareas pendientes. Cuando se marca la casilla «Mostrar solo tareas activas», las tareas completadas no se muestran en la lista. Independientemente de las tareas que sean visibles, el pie de página muestra la cantidad de tareas que aún no han sido completadas.

Simplifica este componente eliminando todo el estado y los Efectos innecesarios.

**App.jsx**
```jsx
import { useState, useEffect } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [activeTodos, setActiveTodos] = useState([]);
  const [visibleTodos, setVisibleTodos] = useState([]);
  const [footer, setFooter] = useState(null);

  useEffect(() => { // 👈
    setActiveTodos(todos.filter(todo => !todo.completed));
  }, [todos]);

  useEffect(() => { // 👈
    setVisibleTodos(showActive ? activeTodos : todos);
  }, [showActive, todos, activeTodos]);

  useEffect(() => { // 👈
    setFooter(
      <footer>
        {activeTodos.length} tareas restantes
      </footer>
    );
  }, [activeTodos]);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Mostrar solo tareas activas
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      {footer}
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Agregar
      </button>
    </>
  );
}
```

**todos.js**
```js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Comprar manzanas', true),
  createTodo('Comprar naranjas', true),
  createTodo('Comprar zanahorias'),
];
```

Muestra:

![[4.1-quiza-no-necesites-un-efecto-1.png]]

Al pulsar la casilla sólo se muestran las tareas activas:

![[4.1-quiza-no-necesites-un-efecto-2.png]]

### Respuesta

Solo hay dos piezas esenciales de estado en este ejemplo: la lista de `todos` y la variable de estado `showActive` que representa si la casilla de verificación está marcada. **Todas las demás variables de estado son [redundantes](https://es.react.dev/learn/choosing-the-state-structure#avoid-redundant-state) y se pueden calcular durante el renderizado. Esto incluye el `footer` que puedes mover directamente al JSX que lo rodea**.

Tu resultado debería verse así:

**App.jsx**
```jsx
import { useState, useEffect } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);

  const activeTodos = todos.filter(todo => !todo.completed) // 👈
  const visibleTodos = showActive ? activeTodos : todos // 👈

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Mostrar solo tareas activas
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      <footer> {/*👈*/}
        {activeTodos.length} tareas restantes {/*👈*/}
      </footer>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Agregar
      </button>
    </>
  );
}
```

## Desafío 2 de 4: Cachear un cálculo sin usar Efectos

En este ejemplo, se extrajo el filtrado de las tareas en una función separada llamada `getVisibleTodos()`. Esta función contiene una llamada a `console.log()` que te ayuda a darte cuenta de cuándo se llama. Alterna «Mostrar solo tareas activas» y observa que esto causa que `getVisibleTodos()` se vuelva a ejecutar. Esto es esperado porque las tareas visibles cambian cuando alternas cuáles se deben mostrar.

Tu tarea es eliminar el Efecto que recalcula la lista de `visibleTodos` en el componente `TodoList`. Sin embargo, debes asegurarte de que `getVisibleTodos()` **no** se vuelva a ejecutar (y, por lo tanto, no muestre ningún registro) cuando escribas en el campo de texto.

**App.jsx**
```jsx
import { useState, useEffect } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const [visibleTodos, setVisibleTodos] = useState([]);

  useEffect(() => {
    setVisibleTodos(getVisibleTodos(todos, showActive));
  }, [todos, showActive]);

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Mostrar solo tareas activas
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Agregar
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

**todo.js**
```js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() se llamó ${++calls} veces`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Comprar manzanas', true),
  createTodo('Comprar naranjas', true),
  createTodo('Comprar zanahorias'),
];
```

Muestra:

![[4.1-quiza-no-necesites-un-efecto-3.png]]

Al marcar y desmarcar la casilla aparecen dos  nuevos mensajes en la consola:

![[4.1-quiza-no-necesites-un-efecto-4.png]]

### Respuesta

Elimina la variable de estado y el Efecto, y en su lugar, agrega una llamada a `useMemo` para cachear el resultado de la función `getVisibleTodos()`:

**App.jsx**
```jsx
import { useState, useMemo } from 'react'; // 👈
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const visibleTodos = useMemo( // 👈
    () => getVisibleTodos(todos, showActive),
    [todos, showActive]
  );

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Mostrar solo tareas activas
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Agregar
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

**todo.js**
```js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() se llamó ${++calls} veces`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Comprar manzanas', true),
  createTodo('Comprar naranjas', true),
  createTodo('Comprar zanahorias'),
];
```

Con este cambio, `getVisibleTodos()` solo se llamará si cambian todos o `showActive`. Escribir en el campo de texto solo cambia la variable de estado text, por lo que no provoca una llamada a `getVisibleTodos()`.

**También una mejor solución que no necesita `useMemo`**. Dado que la variable de estado `text` no puede afectar la lista de tareas, puedes extraer el formulario `NewTodo` en un componente separado y mover la variable de estado `text` dentro de él:

**App.jsx**
```jsx
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const visibleTodos = getVisibleTodos(todos, showActive); // 👈

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Mostrar solo tareas activas
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} /> // 👈
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Agregar
      </button>
    </>
  );
}
```

Este enfoque también cumple con los requisitos. Cuando escribes en el campo de texto, solo se actualiza la variable de estado `text`. Dado que la variable de estado `text` se encuentra en el componente secundario `NewTodo`, el componente padre `TodoList` no se volverá a renderizar. Por eso `getVisibleTodos()` no se llama cuando escribes en el campo de texto (aunque aún se llamaría si `TodoList` se volviera a renderizar por otro motivo).

## Desafío 3 de 4: Reiniciar el estado sin Efectos

Este componente `EditContact` recibe un objeto de contacto con la forma `{ id, name, email }` como _prop_ `savedContact`. Intenta editar los campos de nombre y correo electrónico. Cuando presiones Guardar, el botón del contacto sobre el formulario se actualizará con el nombre editado. Cuando presiones Reiniciar, cualquier cambio pendiente en el formulario se descartará. Juega con esta interfaz para familiarizarte con ella.

Cuando seleccionas un contacto con los botones en la parte superior, el formulario se reinicia para reflejar los detalles de ese contacto. Esto se hace con un Efecto dentro de `EditContact.js`. Elimina este Efecto. Encuentra otra forma de reiniciar el formulario cuando cambie `savedContact.id`.

**EditContact.jsx**
```jsx
import { useState, useEffect } from 'react';

export default function EditContact({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  useEffect(() => {
    setName(savedContact.name);
    setEmail(savedContact.email);
  }, [savedContact]);

  return (
    <section>
      <label>
        Nombre:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Correo electrónico:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Guardar
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reiniciar
      </button>
    </section>
  );
}
```

Muestra:

![[4.1-quiza-no-necesites-un-efecto-5.png]]

Al cambiar de contacto, la información de los inputs se actualiza:

![[4.1-quiza-no-necesites-un-efecto-6.png]]

### Respuesta

Divide el componente `EditContact` en dos. Mueve todo el estado del formulario al componente interno `EditForm`. Exporta el componente externo `EditContact` y haz que pase `savedContact.id` como la _`key`_ al componente interno `EditForm`. Como resultado, el componente interno `EditForm` reiniciará todo el estado del formulario y recreará el DOM cada vez que selecciones un contacto diferente.

**EditForm.jsx**
```jsx
import { useState } from 'react';

export default function EditContact(props) { // 👈
  return (
    <EditForm
      {...props}
      key={props.savedContact.id} // 👈
    />
  );
}

function EditForm({ savedContact, onSave }) { // 👈
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  return (
    <section>
      <label>
        Nombre:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Correo electrónico:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Guardar
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reiniciar
      </button>
    </section>
  );
}
```

Una solución más simple, sería simplemente agregar una key con `id` del contacto a la instancia del componente, para así React pueda diferenciarlos:

**App.jsx**
```jsx
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        savedContact={selectedContact}
        onSave={handleSave}
        key={selectedContact.id} // 👈
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

**EditForm.jsx**
```jsx
import { useState, useEffect } from 'react';

export default function EditContact({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  // useEffect(() => { // 👈❌
  //   setName(savedContact.name);
  //   setEmail(savedContact.email);
  // }, [savedContact]);

  return (
    <section>
      <label>
        Nombre:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Correo electrónico:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Guardar
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reiniciar
      </button>
    </section>
  );
}
```

