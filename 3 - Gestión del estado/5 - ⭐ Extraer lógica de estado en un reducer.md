Los componentes con muchas actualizaciones de estado distribuidas a través de varios manejadores de eventos pueden ser agobiantes. Para estos casos, **puedes consolidar toda la lógica de actualización de estado fuera del componente en una única función, llamada _reducer._**

### Aprenderás

- Qué es una función de reducer
- Cómo refactorizar de `useState` a `useReducer`
- Cuándo utilizar un reducer
- Cómo escribir uno de manera correcta

## ⭐ Consolidar lógica de estado con un _reducer_

A medida que tus componentes crecen en complejidad, puede volverse difícil seguir a simple vista todas las formas en que el estado de un componente se actualiza. Por ejemplo, el componente `TaskApp` mantiene un _array_ de `tasks` (tareas) en el estado y usa tres manejadores de eventos diferentes para agregar, borrar y editar tareas.

**App.js**
```jsx
import { useState } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) { // 👈
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) { // 👈
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) { // 👈
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>Itinerario en Praga</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visitar el Museo Kafka', done: true},
  {id: 1, text: 'Ver espectáculo de títeres', done: false},
  {id: 2, text: 'Foto del muro de Lennon', done: false},
];
```

Muestra:

![[5-extraer-lógica-de-estado-en-un-reducer-2.png]]

Cada uno de estos manejadores de eventos llama a `setTasks` con el fin de actualizar el estado. A medida que el componente crece, también lo hace la cantidad de lógica de estado esparcida a lo largo de este. **Para reducir esta complejidad y mantener toda la lógica en un lugar de fácil acceso, puedes mover esa lógica de estado a una función única fuera del componente llamada un «_reducer_».**

Los _reducers_ son una forma diferente de manejar el estado. Puedes migrar de `useState` a `useReducer` en tres pasos:

1. **Cambia** de asignar un estado a despachar acciones.
2. **Escribe** una función _reducer_.
3. **Usa** el _reducer_ desde tu componente.

### Paso 1: Cambia de establecer un estado a despachar acciones 

Tus manejadores de eventos actualmente especifican _qué hacer_ al asignar el estado:

```jsx
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

**Elimina toda la lógica de asignación de estado. Lo que queda son estos tres manejadores de eventos**:

- `handleAddTask(text)` se llama cuando el usuario presiona «Agregar».
- `handleChangeTask(task)` se llama cuando el usuario cambia una tarea o presiona «Guardar».
- `handleDeleteTask(taskId)` se llama cuando el usuario presiona «Borrar».

Manejar el estado con reducers es ligeramente diferente a asignar directamente el estado. En lugar de decirle a React «_qué hacer_» al asignar el estado, especificas «_qué acaba de hacer el usuario_» despachando «_acciones_» desde tus manejadores de eventos. (¡La lógica de actualización de estado estará en otro lugar!) Entonces, **en lugar de «_asignar `tasks`_» a través de un manejador de evento, estás despachando una acción de «_tarea agregada/cambiada/borrada (added/changed/deleted_)». Esta forma describe más la intención del usuario**.

```jsx
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
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

El objeto que pasas a `dispatch` se denomina «acción»:

```jsx
function handleDeleteTask(taskId) {
  dispatch(
    // objeto "acción":
    {
      type: 'deleted',
      id: taskId,
    }
  );
}
```

Es un objeto regular de JavaScript. Tú decides qué poner dentro, pero generalmente debe contener la mínima información acerca de _qué ocurrió_. (Agregarás la función `dispatch` en un paso posterior).

> [!note]
>Un objeto de acción puede tener cualquier forma.
>
>Por convención, es común proporcionar un texto `type` que describe qué ocurrió, y transmite cualquier información adicional en otros campos. El `type` es específico a un componente, en este ejemplo tanto `'added'` o `'added_task'` estaría bien. ¡Elige un nombre que describa que ocurrió!
> ```jsx
> dispatch({
>   // específico al componente
>   type: 'what_happened',
>   // otros campos van aquí
> });
> ```

### Paso 2: Escribe una función reducer 

Una función reducer es donde pondrás tu lógica de estado. **Recibe dos argumentos, el estado actual y el objeto de acción, y devuelve el próximo estado**.

```js
function yourReducer(state, action) {
  // devuelve el próximo estado para que React lo asigne
}
```

**React asignará el estado a lo que se devuelve desde el reducer**.

Para mover la lógica de asignación de estado desde tus manejadores de eventos a una función reducer en este ejemplo, vas a:

1. Declarar el estado actual (`tasks`) como primer argumento.
2. Declarar el objeto `action` como segundo argumento.
3. Devolver el _próximo_ estado desde el reducer (con el cual React asignará el estado).

Aquí se encuentra toda la lógica de asignación de estado migrada a una función reducer:

```jsx
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
```

> Como la función reducer recibe el estado (`tasks`) como un argumento, puedes **declararlo fuera de tu componente.** Esto reduce el nivel de tabulación y puede hacer que tu código sea más fácil de leer.

> [!tip]
>
>El código de arriba usa esta sentencias if/else, pero es una convención usar [sentencias switch](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Statements/switch) dentro de los reducers. El resultado es el mismo, pero puede ser más sencillo leer sentencias switch a simple vista.
>
>Las estaremos usando en el resto de esta documentación de la siguiente manera:
>
>```jsx
>function tasksReducer(tasks, action) {
>  switch (action.type) {
>    case 'added': {
>      return [
>        ...tasks,
>        {
>          id: action.id,
>          text: action.text,
>          done: false,
>        },
>      ];
>    }
>    case 'changed': {
>      return tasks.map((t) => {
>        if (t.id === action.task.id) {
>          return action.task;
>        } else {
>          return t;
>        }
>      });
>    }
>    case 'deleted': {
>      return tasks.filter((t) => t.id !== action.id);
>    }
>    default: {
>      throw Error('Unknown action: ' + action.type);
>    }
>  }
>}
>```
>
>Recomendamos envolver cada instancia `case` entre llaves `{` y `}`, así las variables declaradas dentro de los diferentes `case`s no entran en conflicto. También, un `case` debería generalmente terminar con un `return`. ¡Si olvidas hacer un `return`, el código «caerá» hasta el siguiente `case`, lo que puede llevarte a errores!
>
>Si todavía no te sientes cómodo con las sentencias switch, está bien usar if/else.

#### ¿Por qué los reducers se llaman de esta manera?

Aunque los reducers pueden «_reducir_» la cantidad de código dentro de tu componente, **son en realidad llamados así por la operación [`reduce()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) la cual se puede realizar en arrays**.

La operación `reduce()` permite tomar un array y «acumular» un único valor a partir de varios:

```js
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce(
  (result, number) => result + number
); // 1 + 2 + 3 + 4 + 5
```

La función que se pasa para `reducir` es conocida como un «?». Toma el _resultado hasta el momento_ y el _elemento actual,_ luego devuelve el _siguiente resultado._ Los reducers de React son un ejemplo de la misma idea: **toman el _estado hasta el momento_ y la _acción_, y devuelven el _siguiente estado._ De esta manera, se acumulan acciones sobre el tiempo en estado**.

Puedes incluso utilizar el método `reduce()` con un estado inicial (`initialState`) y un array de acciones (`actions`) para calcular el estado final pasándole tu función reducer:

**index.js**
```jsx
import tasksReducer from './tasksReducer.js';

let initialState = [];
let actions = [
  {type: 'added', id: 1, text: 'Visitar el Museo Kafka'},
  {type: 'added', id: 2, text: 'Ver espectáculo de títeres'},
  {type: 'deleted', id: 1},
  {type: 'added', id: 3, text: 'Foto del muro de Lennon'},
];

let finalState = actions.reduce(tasksReducer, initialState);

const output = document.getElementById('output');
output.textContent = JSON.stringify(finalState, null, 2);
```

**index.html**
```jsx
<pre id="output"></pre>
```

**tasksReducer.js**
```jsx
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

Muestra:

![[5-extraer-lógica-de-estado-en-un-reducer-3.png]]

Probablemente no necesites hacer esto por tu cuenta, pero ¡es similar a lo que hace React!
### Paso 3: Usa el reducer desde tu componente 

Finalmente, debes conectar el `tasksReducer` a tu componente. Asegúrate de importar el Hook `useReducer` de React:

```jsx
import { useReducer } from 'react';
```

Luego puedes reemplazar `useState`:

```jsx
const [tasks, setTasks] = useState(initialTasks);
```

con `useReducer` de esta manera:

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

**El Hook `useReducer` es similar a `useState`; debes pasar un estado inicial y devuelve un valor de estado y una manera de actualizar estado (en este caso, la función dispatch)**. Pero es un poco diferente.

El Hook `useReducer` toma dos parámetros:

1. Una función reducer
2. Un estado inicial

Y devuelve:

1. Un valor de estado
2. Una función dispatch (para «despachar» acciones del usuario hacia el reducer)

¡Ahora está completamente conectado! Aquí, el reducer se declara al final del archivo del componente:

**App.js**
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

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
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Itinerario en Praga</h1>
      <AddTask onAddTask={handleAddTask} />
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
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visitar el Museo Kafka', done: true},
  {id: 1, text: 'Ver espectáculo de títeres', done: false},
  {id: 2, text: 'Foto del muro de Lennon', done: false},
];
```

Muestra:

![[5-extraer-lógica-de-estado-en-un-reducer-2.png]]


Si lo deseas, puedes incluso mover el reducer a un archivo diferente:

**App.js**
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import tasksReducer from './tasksReducer.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

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
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Itinerario en Praga</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visitar el Museo Kafka', done: true},
  {id: 1, text: 'Ver espectáculo de títeres', done: false},
  {id: 2, text: 'Foto del muro de Lennon', done: false},
];
```

**tasksReducer.js**
```jsx
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

La lógica del componente puede ser más sencilla de leer cuando separas conceptos como este. Ahora los manejadores de eventos sólo especifican _qué ocurrió_ despachando acciones, y la función reducer determina _cómo se actualiza el estado_ en respuesta a ellas.

## Comparación de `useState` y `useReducer`

¡Los reducers no carecen de desventajas! Aquí hay algunas maneras en las que puedas compararlos:

- **Tamaño del código:** Generalmente, con `useState` debes escribir menos código por adelantado. Con `useReducer`, debes escribir la función de reducer _y_ las actions a despachar. Sin embargo, `useReducer` puede ayudar a disminuir el código si demasiados manejadores de eventos modifican el estado de una manera similar
- **Legibilidad:** `useState` es muy sencillo de leer cuando las actualizaciones de estado son simples. Cuando se vuelven más complejas, pueden inflar el código de tu componente y hacerlo difícil de escanear. En este caso, `useReducer` te permite separar limpiamente el _cómo_ de la lógica de actualización del _que ocurrió_ de los manejadores de eventos.
- **Depuración:** Cuando tienes un error con `useState`, puede ser difícil decir _dónde_ el estado se ha actualizado incorrectamente, y _por qué_. Con `useReducer`, puedes agregar un _log_ de la consola en tu reducer para ver cada actualización de estado, y _por qué_ ocurrió (debido a qué acción). Si cada acción es correcta, sabrás que el error se encuentra en la propia lógica del reducer. Sin embargo, debes pasar por más código que con `useState`.
- **Pruebas:** Un reducer es una función pura que no depende de tu componente. Esto significa que puedes exportarla y probarla separadamente de manera aislada. Mientras que generalmente es mejor probar componentes en un entorno más realista, para actualizaciones de estado complejas, puede ser útil asegurar que tu reducer devuelve un estado particular para un estado y acción particular.
- **Preferencia personal:** Algunas personas prefieren reducers, otras no. Está bien. Es una cuestión de preferencia. Siempre puedes convertir entre `useState` y `useReducer` de un lado a otro: ¡son equivalentes!

Recomendamos utilizar un reducer si a menudo encuentras errores debidos a actualizaciones incorrectas de estado en algún componente, y deseas introducir más estructura a tu código. No es necesario usar reducers para todo: ¡siente la libertad de mezclar y combinar! Incluso puedes tener `useState` y `useReducer` en el mismo componente.

## Escribir reducers correctamente 

Ten en cuenta estos dos consejos al escribir reducers:

- **Los reducers deben ser puros.** Al igual que las [funciones de actualización de estado](https://es.react.dev/learn/queueing-a-series-of-state-updates), los reducers ¡se ejecutan durante el renderizado! (Las _actions_ se ponen en cola hasta el siguiente renderizado) Esto significa que los reducers [deben ser puros](https://es.react.dev/learn/keeping-components-pure) —la misma entrada siempre produce el mismo resultado—. No deben enviar peticiones de red, programar _timeouts_, o realizar ningún tipo de _efecto secundario_ (operaciones con impacto fuera del componente). Deben actualizar [objetos](https://es.react.dev/learn/updating-objects-in-state) y [arrays](https://es.react.dev/learn/updating-arrays-in-state) sin mutaciones.
- **Cada acción describe una única interacción del usuario, incluso si eso conduce a múltiples cambios en los datos.** Por ejemplo, si un usuario presiona «Reiniciar» en un formulario con cinco campos manejados por un reducer, tiene más sentido despachar una acción `reset_form` en lugar de cinco acciones de `set_field`. Si registras cada acción en un reducer, ese registro debería ser suficientemente claro como para reconstruir qué interacciones o respuestas pasaron y en que orden. ¡Esto ayuda en la depuración!

## ⭐ Escribir reducers concisos con Immer 

Al igual que para [actualizar objetos](https://es.react.dev/learn/updating-objects-in-state#write-concise-update-logic-with-immer) y [arrays](https://es.react.dev/learn/updating-arrays-in-state#write-concise-update-logic-with-immer), para el estado regular se puede utilizar la biblioteca Immer para hacer los reducers más concisos. Aquí, [`useImmerReducer`](https://github.com/immerjs/use-immer#useimmerreducer) te permite mutar el estado con `push` o una asignación `arr[i] =`:

**App.js**
```jsx
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

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
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Itinerario en Praga</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visitar el Museo Kafka', done: true},
  {id: 1, text: 'Ver espectáculo de títeres', done: false},
  {id: 2, text: 'Foto del muro de Lennon', done: false},
];
```


Los reducers deben ser puros, así que no deberían mutar estado. Pero Immer te proporciona un objeto especial `draft` que se puede mutar con seguridad. Por detrás, Immer creará una copia de tu estado con los cambios que has hecho a este objeto `draft`. Esta es la razón por la que los reducers manejados con `useImmerReducer` pueden mutar su primer argumento y no necesitan devolver un estado.

## Recapitulación

- Para convertir de `useState` a `useReducer`:
    1. Despacha acciones desde manejadores de eventos.
    2. Escribe una función reducer que devuelve el siguiente estado para un estado y acción dados.
    3. Reemplaza `useState` con `useReducer`.
- Los reducers requieren que escribas un poco más de código, pero ayudan con la depuración y las pruebas.
- Los reducers deben ser puros.
- Cada acción describe una interacción única del usuario.
- Usa Immer si deseas escribir reducers como si se estuviera mutando el estado.