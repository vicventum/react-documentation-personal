Los _reducers_ permiten consolidar la lógica de actualización del estado de un componente. La API de contexto (Context) te permite pasar información en profundidad a otros componentes. **Puedes combinar los _reducers_ y el contexto para gestionar el estado de una pantalla compleja**.

### Aprenderás

- Cómo combinar un _reducer_ con el contexto
- Cómo evitar pasar el estado y la función _dispatch_ a través de props
- Cómo mantener la lógica del contexto y del estado en un archivo separado

## Combinar un _reducer_ con el contexto

En este ejemplo de [Introducción a _reducers_](https://es.react.dev/learn/extracting-state-logic-into-a-reducer), el estado es gestionado por un _reducer_. La función _reducer_ contiene toda la lógica de actualización del estado y se declara al final de este archivo:

**App.jsx**
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Día libre en Kyoto</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'El Camino del Filósofo', done: true },
  { id: 1, text: 'Visitar el templo', done: false },
  { id: 2, text: 'Beber té matcha', done: false }
];
```

**AddTask.js**
```jsx
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Agregar tarea"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Agregar</button>
    </>
  )
}
```

**TaskList.js**
```jsx
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Guardar
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Editar
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Borrar
      </button>
    </label>
  );
}
```

Muestra:

![[7-escalar-con-reducer-y-context-1.png]]

Un _reducer_ ayuda a mantener los controladores de eventos cortos y concisos. **Sin embargo, a medida que tu aplicación crece, puedes encontrarte con otra dificultad. Actualmente, el estado `tasks` y la función `dispatch` sólo están disponibles en el componente de nivel superior `TaskApp`. Para permitir que otros componentes lean la lista de tareas o la modifiquen, tienes que [pasar](https://es.react.dev/learn/passing-props-to-a-component) explícitamente el estado actual y los controladores de eventos que lo cambian como props**.

Por ejemplo, `TaskApp` pasa una lista de tareas y los controladores de eventos a `TaskList`:

```jsx
<TaskList
  tasks={tasks}
  onChangeTask={handleChangeTask}
  onDeleteTask={handleDeleteTask}
/>
```

Y `TaskList` pasa los controladores de eventos a `Task`:

```jsx
<Task
  task={task}
  onChange={onChangeTask}
  onDelete={onDeleteTask}
/>
```

En un ejemplo pequeño como éste, funciona bien, pero si tienes decenas o cientos de componentes en el medio, ¡pasar todo el estado y las funciones puede ser bastante frustrante!

Por eso, como alternativa a pasarlas por props, **podrías poner tanto el estado `tasks` como la función `dispatch` [en el contexto.](https://es.react.dev/learn/passing-data-deeply-with-context) De esta manera, cualquier componente por debajo de `TaskApp` en el árbol puede leer las tareas y enviar acciones sin la «perforación de props» (o _«prop drilling»_)**.

A continuación se explica cómo se puede combinar un _reducer_ con el contexto:

1. **Crea** el contexto.
2. **Pon** el estado y la función _dispatch_ en el contexto.
3. **Usa** el contexto en cualquier parte del árbol.

### Paso 1: Crea el contexto

El hook `useReducer` devuelve las tareas actuales y la función `dispatch` que permite actualizarlas:

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

Para pasarlos por el árbol, [crearás](https://es.react.dev/learn/passing-data-deeply-with-context#step-2-use-the-context) dos contextos distintos:

- `TasksContext` proporciona la lista actual de tareas.
- `TasksDispatchContext` proporciona la función que permite a los componentes enviar acciones.

Expórtalos desde un archivo separado para poder importarlos posteriormente desde otros archivos:

**App.jsx**
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Día libre en Kyoto</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'El Camino del Filósofo', done: true },
  { id: 1, text: 'Visitar el templo', done: false },
  { id: 2, text: 'Beber té matcha', done: false }
];
```

**TasksContext.js**
```js
import { createContext } from 'react'; // 👈

export const TasksContext = createContext(null); // 👈
export const TasksDispatchContext = createContext(null); // 👈
```

**AddTask.js**
```jsx
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Agregar tarea"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Agregar</button>
    </>
  )
}
```

**TaskList.js**
```jsx
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Guardar
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Editar
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Borrar
      </button>
    </label>
  );
}
```

Muestra:

![[7-escalar-con-reducer-y-context-1.png]]

Aquí, estás pasando `null` como valor por defecto a ambos contextos. Los valores reales serán proporcionados por el componente `TaskApp`.

### Paso 2: Poner en contexto el estado y _dispatch_ 

Ahora puedes importar ambos contextos en tu componente `TaskApp`. Toma `tasks` y `dispatch` que devuelve `useReducer()` y [proporciónalos](https://es.react.dev/learn/passing-data-deeply-with-context#step-3-provide-the-context) a todo el árbol de abajo::

**App.jsx**
```jsx
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks); // 👈
  // ...
  return (
    <TasksContext.Provider value={tasks}> // 👈
      <TasksDispatchContext.Provider value={dispatch}> // 👈
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

Por ahora, se pasa la información tanto vía props como en contexto:

**App.jsx**
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksContext, TasksDispatchContext } from './TasksContext.js'; // 👈

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <TasksContext.Provider value={tasks}> // 👈
      <TasksDispatchContext.Provider value={dispatch}> // 👈
        <h1>Día libre en Kyoto</h1>
        <AddTask
          onAddTask={handleAddTask}
        />
        <TaskList
          tasks={tasks}
          onChangeTask={handleChangeTask}
          onDeleteTask={handleDeleteTask}
        />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'El Camino del Filósofo', done: true },
  { id: 1, text: 'Visitar el templo', done: false },
  { id: 2, text: 'Beber té matcha', done: false }
];

```

**TasksContext.js**
```js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

**AddTask.js**
```jsx
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Agregar tarea"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Agregar</button>
    </>
  )
}
```

**TaskList.js**
```jsx
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Guardar
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Editar
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Borrar
      </button>
    </label>
  );
}
```

Muestra:

![[7-escalar-con-reducer-y-context-1.png]]

En el siguiente paso, se eliminará el paso de props.

### Paso 3: Utiliza el contexto en cualquier parte del árbol

Ahora no es necesario pasar la lista de tareas o los controladores de eventos por el árbol:

**App.jsx**
```jsx
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    <h1>Día libre en Kyoto</h1>
    <AddTask /> // 👈
    <TaskList /> // 👈
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
```

En cambio, cualquier componente que necesite la lista de tareas puede leerla del `TaskContext`:

```jsx
export default function TaskList() {
  const tasks = useContext(TasksContext); // 👈
  // ...
```

Para actualizar la lista de tareas, cualquier componente puede leer la función `dispatch` del contexto y llamarla:

```jsx
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext); // 👈
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({ // 👈
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Agregar</button>
    // ...
```

**El componente `TaskApp` no pasa ningún controlador de evento hacia abajo, y `TaskList` tampoco pasa ningún controlador de evento al componente `Task`.** Cada componente lee el contexto que necesita:

**App.jsx**
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Día libre en Kyoto</h1>
        <AddTask />
        <TaskList />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'El Camino del Filósofo', done: true },
  { id: 1, text: 'Visitar el templo', done: false },
  { id: 2, text: 'Beber té matcha', done: false }
];
```

**TasksContext.js**
```js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

**AddTask.js**
```jsx
import { useState, useContext } from 'react';
import { TasksDispatchContext } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  return (
    <>
      <input
        placeholder="Agregar tarea"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        }); 
      }}>Agregar</button>
    </>
  );
}

let nextId = 3;
```

**TaskList.js**
```jsx
import { useState, useContext } from 'react';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskList() {
  const tasks = useContext(TasksContext);
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useContext(TasksDispatchContext);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Guardar
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Editar
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({
          type: 'deleted',
          id: task.id
        });
      }}>
        Borrar
      </button>
    </label>
  );
}
```

Muestra:

![[7-escalar-con-reducer-y-context-1.png]]

**El estado todavía «vive» en el componente de nivel superior `TaskApp`, gestionado con `useReducer`.** Pero sus tareas (`tasks`) y `dispatch` están ahora disponibles para todos los componentes por debajo en el árbol mediante la importación y el uso de estos contextos.

## Trasladar todo la lógica a un único archivo

No es necesario que lo hagas, pero podrías simplificar aún más los componentes moviendo tanto el _reducer_ como el contexto a un solo archivo. Actualmente, `TasksContext.js` contiene solo dos declaraciones de contexto:

**TasksContext.js**
```js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

¡Este archivo está a punto de complicarse! Moverás el _reducer_ a ese mismo archivo. A continuación, declararás un nuevo componente `TasksProvider` en el mismo archivo. Este componente unirá todas las piezas:

1. Gestionará el estado con un _reducer_.
2. Proporcionará ambos contextos a los componentes de abajo.
3. [Tomará `children` como prop](https://es.react.dev/learn/passing-props-to-a-component#passing-jsx-as-children) para que puedas pasarle JSX.

```jsx
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

**Esto elimina toda la complejidad y la lógica del componente `TaskApp`:**

**App.jsx**
```jsx
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Día libre en Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

**TasksContext.js**
```jsx
import { createContext, useReducer } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'El Camino del Filósofo', done: true },
  { id: 1, text: 'Visitar el templo', done: false },
  { id: 2, text: 'Beber té matcha', done: false }
];
```

**AddTask.js**
```jsx
import { useState, useContext } from 'react';
import { TasksDispatchContext } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  return (
    <>
      <input
        placeholder="Agregar tarea"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        }); 
      }}>Agregar</button>
    </>
  );
}

let nextId = 3;
```

**TaskList.js**
```jsx
import { useState, useContext } from 'react';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskList() {
  const tasks = useContext(TasksContext);
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useContext(TasksDispatchContext);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Guardar
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Editar
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({
          type: 'deleted',
          id: task.id
        });
      }}>
        Borrar
      </button>
    </label>
  );
}
```

Muestra:

![[7-escalar-con-reducer-y-context-1.png]]  

También puedes exportar funciones que _utilicen_ el contexto desde `TasksContext.js`:

```js
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

Cuando un componente necesita leer el contexto, puede hacerlo a través de estas funciones:

```js
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

Esto no cambia el comportamiento de ninguna manera, pero te permite dividir más tarde estos contextos o añadir algo de lógica a estas funciones. **Ahora todo la lógica del contexto y del reducer está en `TasksContext.js`. Esto mantiene los componentes limpios y despejados, centrados en lo que muestran en lugar de donde obtienen los datos:**

**App.jsx**
```jsx
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Día libre en Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

**TasksContext.jsx**
```jsx
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);

const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

export function useTasks() { // 👈
  return useContext(TasksContext);
}

export function useTasksDispatch() { // 👈
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'El Camino del Filósofo', done: true },
  { id: 1, text: 'Visitar el templo', done: false },
  { id: 2, text: 'Beber té matcha', done: false }
];
```

**AddTask.js**
```jsx
import { useState } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();
  return (
    <>
      <input
        placeholder="Agregar tarea"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        }); 
      }}>Agregar</button>
    </>
  );
}

let nextId = 3;
```

**TaskList.jsx**
```jsx
import { useState } from 'react';
import { useTasks, useTasksDispatch } from './TasksContext.js';

export default function TaskList() {
  const tasks = useTasks();
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useTasksDispatch();
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Guardar
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Editar
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({
          type: 'deleted',
          id: task.id
        });
      }}>
        Borrar
      </button>
    </label>
  );
}

```

Muestra:

![[7-escalar-con-reducer-y-context-1.png]]  

Puedes pensar en `TasksProvider` como una parte de la pantalla que sabe cómo tratar con las tareas, `useTasks` como una forma de leerlas, y `useTasksDispatch` como una forma de actualizarlas desde cualquier componente de abajo en el árbol.

> [!info]
> Funciones como `useTasks` y `useTasksDispatch` se llaman **[Hooks personalizados _(Custom Hooks)_.](https://es.react.dev/learn/reusing-logic-with-custom-hooks)** Tu función se considera un Hook personalizado si su nombre empieza por `use`. Esto te permite usar otros Hooks, como `useContext`, dentro de ella.

A medida que tu aplicación crece, puedes tener muchos pares contexto-_reducer_ como este. Esta es una poderosa forma de escalar tu aplicación y [manejar el estado](https://es.react.dev/learn/sharing-state-between-components) sin demasiado trabajo cada vez que se quiera acceder a los datos en la profundidad del árbol.

## Recapitulación

- Puedes combinar el reducer con el contexto para permitir que cualquier componente lea y actualice el estado por encima de él.
- Para proporcionar estado y la función _dispatch_ a los componentes de abajo:
    1. Crea dos contextos (para el estado y para las funciones _dispatch_).
    2. Proporciona ambos contextos desde el componente que utiliza el reducer.
    3. Utiliza cualquiera de los dos contextos desde los componentes que necesiten leerlos.
- Puedes refactorizar aún más los componentes moviendo todo la lógica a un solo archivo.
    - Puedes exportar un componente como `TasksProvider` que proporciona el contexto.
    - También puedes exportar Hooks personalizados como `useTasks` y `useTasksDispatch` para leerlo.
- Puedes tener muchos pares context-_reducer_ como este en tu aplicación.