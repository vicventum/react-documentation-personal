React viene con varios Hooks integrados como `useState`, `useContext` y `useEffect`. **A veces, desearás que haya un Hook para algún propósito más específico**: por ejemplo, para obtener datos, para mantener un seguimiento de si un usuario está conectado o para conectarse a una sala de chat. **Es posible que no encuentres estos Hooks en React, pero puedes crear tus propios Hooks para las necesidades de tu aplicación**.

### Aprenderás

- Qué son los Hooks personalizados y cómo escribirlos por tu cuenta
- Cómo reutilizar lógica entre componentes
- Cómo nombrar y estructurar tus Hooks personalizados
- Cuándo y por qué extraer Hooks personalizados

## ⭐ _Custom Hooks_: compartir lógica entre componentes

Imagina que estás desarrollando una aplicación que depende en gran medida de la red (como la mayoría de aplicaciones). Quieres informar al usuario si su conexión de red se interrumpió accidentalmente mientras usaba tu aplicación. ¿Cómo lo harías? Parece que necesitarás dos cosas en tu componente:

1. Una parte del estado que rastree si la red está en línea.
2. Un Efecto que se suscriba a los eventos globales: [`online`](https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event) y [`offline`](https://developer.mozilla.org/en-US/docs/Web/API/Window/offline_event), y actualice ese estado.

Esto mantendrá tu componente [sincronizado](https://es.react.dev/learn/synchronizing-with-effects) con el estado de la red. Podrías empezar con algo como esto:

**App.js**
```jsx
import { useState, useEffect } from 'react';

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return <h1>{isOnline ? '✅ En línea' : '❌ Desconectado'}</h1>;
}
```

Muestra:

![[8-reutilizar-la-logica-con-custom-hooks-1.png]]

Intenta conectar y desconectar la red y observa cómo este `StatusBar` se actualiza en respuesta a tus acciones.

Ahora imagina que _también_ quieres usar la misma lógica en un componente diferente. Quieres implementar un botón Guardar que se desactivará y mostrará «Reconectando…» en lugar de «Guardar» mientras la red está desconectada.

Para empezar, puedes copiar y pegar el estado `isOnline` y el Efecto en `SaveButton`:

**App.js**
```jsx
import { useState, useEffect } from 'react';

export default function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  function handleSaveClick() {
    console.log('✅ Progreso guardado');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Guardar progreso' : 'Reconectando...'}
    </button>
  );
}
```
Muestra:

![[8-reutilizar-la-logica-con-custom-hooks-2.png]]

Al presionar sobre el botón, se muestra un mensaje por consola:

![[8-reutilizar-la-logica-con-custom-hooks-3.png]]

Comprueba que, si apagas la red, el botón cambiará su apariencia.

**Estos dos componentes funcionan bien, pero la duplicación de lógica entre estos es desafortunada**. Parece que a pesar de que tienen una _apariencia visual_ diferente, quieres reutilizar la lógica entre estos.

### ⭐ Extraer tu propio Hook personalizado de un componente 

**Imagina por un momento que, de forma similar a [`useState`](https://es.react.dev/reference/react/useState) y [`useEffect`](https://es.react.dev/reference/react/useEffect), hay un Hook integrado denominado `useOnlineStatus`**. Entonces, ambos componentes podrían simplificarse y podrías eliminar la duplicación entre estos:

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus(); // 👈
  return <h1>{isOnline ? '✅ Conectado' : '❌ Desconectado'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus(); // 👈

  function handleSaveClick() {
    console.log('✅ Progreso guardado');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Guardar progreso' : 'Reconectando...'}
    </button>
  );
}
```

**Aunque no exista un Hook integrado, puedes escribirlo por tu cuenta. Declara una función llamada `useOnlineStatus` y mueve todo el código duplicado dentro, desde los componentes que escribiste anteriormente**:

```jsx
function useOnlineStatus() { // 👈
  const [isOnline, setIsOnline] = useState(true); // 👈
  useEffect(() => { // 👈
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

En el final de la función, devuelve `isOnline`. Esto permite a tus componentes leer ese valor:

**App.js**
```jsx
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Conectado' : '❌ Desconectado'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progreso guardado');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Guardar progreso' : 'Reconectando...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

**useOnlineStatus.js**
```jsx
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

Verifica que al conectar y desconectar la red se actualicen ambos componentes.

Ahora tus componentes no tienen tanta lógica repetitiva. **Más importante aún, el código dentro de estos describe _qué quieren hacer_ (¡usar el estatus de conectado!) en lugar de _cómo hacerlo_ (suscribirse a los eventos del navegador).**

Cuando extraes lógica en un Hook personalizado, puedes ocultar los detalles retorcidos de cómo tratas con algún sistema externo o una API del navegador. El código de tus componentes expresan tu intención, no la implementación.

### Los nombres de Hooks siempre empiezan con `use` 

Las aplicaciones de React están construidas por componentes. Los componentes son construidos a partir de Hooks, ya sean integrados o personalizados. Es probable que a menudo uses Hooks personalizados creados por alguien más, pero ocasionalmente, ¡podrías escribir uno!

Debes seguir estas convenciones de nomenclatura:

1. **Los nombres de componentes de React deben empezar con mayúscula,** como `StatusBar` y `SaveButton`. Los componentes de React también necesitan devolver algo que React conozca cómo mostrar, como un bloque de JSX
2. **Los nombres de Hooks deben empezar con `use` seguido por una mayúscula,** como [`useState`](https://es.react.dev/reference/react/useState) (integrado) o `useOnlineStatus` (personalizado, como antes en la página). Los Hooks pueden devolver valores arbitrarios.

**Esta convención garantiza que siempre puedas mirar un componente y saber dónde pueden «esconder» su estado, Efectos, y otras características de React**. Por ejemplo, si ves una invocación de función denominada `getColor()` en tu componente, puedes estar seguro de que no contiene un estado de React, porque su nombre no empieza con `use`. Sin embargo, una invocación de función como `useOnlineStatus()`, probablemente contendrá otros Hooks.

> [!note]
>Si tu linter está [configurado para React,](https://es.react.dev/learn/editor-setup#linting) hará cumplir esta convención de nomenclatura. Desplázate hasta el sandbox de arriba y cambia el nombre de `useOnlineStatus` a `getOnlineStatus`. Ten en cuenta que el linter ya no permitirá utilizar `useState` o `useEffect` dentro de este nunca más. ¡Únicamente los componentes y Hooks pueden llamar a otros Hooks!

#### ⭐ ¿Deberían comenzar con el prefijo _use_ todas las funciones que se llaman durante el renderizado? 

No. Las funciones que no _utilizan_ Hooks no necesitan _ser_ Hooks.

==**Si tu función no utiliza ningún Hook, evita el prefijo `use`. En su lugar, escríbelo como una función normal _sin_ el prefijo `use`**==. Por ejemplo, a continuación `useSorted` no utiliza Hooks, entonces llámalo como `getSorted`:

```jsx
// 🔴 Evita: Un Hook que no usa Hooks
function useSorted(items) {
  return items.slice().sort();
}

// ✅ Correcto: Una función normal que no usa Hooks
function getSorted(items) {
  return items.slice().sort();
}
```

Esto asegura que tu código puede invocar esta función en cualquier lugar, incluyendo condicionales:

```jsx
function List({ items, shouldSort }) {
  let displayedItems = items;
  if (shouldSort) {
    // ✅ Está bien llamar getSorted() condicionalmente porque no es un Hook
    displayedItems = getSorted(items);
  }
  // ...
}```

**==Deberías colocar el prefijo `use` a una función (y por lo tanto convertirla en un Hook) si usas al menos un Hook dentro de esta==**:

```jsx
// ✅ Correcto: Un Hook que usa otros Hooks
function useAuth() {
  return useContext(Auth);
}
```

Técnicamente, esto no es aplicado por React. En principio, puedes hacer un Hook que no utilice otros Hooks. Esto suele ser confuso y limitante, por lo que es mejor evitar ese patrón. Sin embargo, puede haber casos raros donde sea útil. Por ejemplo, quizás tu función no utilice ningún Hook en este momento, pero planeas añadir algunos Hooks en el futuro. Entonces, tiene sentido nombrarla con prefijo `use`.

```jsx
// ✅ Correcto: Un Hook que probablemente contendrá otros Hooks después
function useAuth() {
  // TODO: Reemplaza con esta línea cuando la autenticación esté implementada:
  // return useContext(Auth);
  return TEST_USER;
}
```

Entonces los componentes no serán capaces de utilizarlo condicionalmente. Esto se volverá importante cuando realmente agregues Hooks dentro. Si no planeas utilizar Hooks dentro (ahora o más tarde), no lo conviertas en un Hook.

### ⭐ Los Hooks personalizados permiten compartir la lógica con estado, no el estado en sí mismo 

En el ejemplo anterior, cuando conectabas y desconectabas la red, ambos componentes se actualizaban juntos. Sin embargo, **es erróneo pensar que una única variable de estado `isOnline` se comparte entre ellos. Mira este código**:

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus(); // 👈
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus(); // 👈
  // ...
}
```

Funciona de la misma manera que antes de extraer la duplicación:

```jsx
function StatusBar() {
  const [isOnline, setIsOnline] = useState(true); // 👈
  useEffect(() => { // 👈
    // ...
  }, []); // 👈
  // ...
}

function SaveButton() {
  const [isOnline, setIsOnline] = useState(true); // 👈
  useEffect(() => { // 👈
    // ...
  }, []); // 👈
  // ...
}
```

**¡Estas son dos variables de estado y Efectos completamente independientes!** Solo tuvieron el mismo valor al mismo tiempo, porque las sincronizaste con el mismo valor externo (si la red está conectada o no).

Para ilustrar mejor esto, necesitaremos un ejemplo diferente. Considera este componente `Form`:

**App.js**
```jsx
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('Mary');
  const [lastName, setLastName] = useState('Poppins');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <label>
        Nombre:
        <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Apellido:
        <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p><b>Buenos días, {firstName} {lastName}.</b></p>
    </>
  );
}
```

Existe cierta lógica repetitiva para cada campo del formulario:

1. Hay un pedazo de estado (`firstName` y `lastName`).
2. Hay una función controladora de cambios (`handleFirstNameChange` y `handleLastNameChange`).
3. Hay un bloque de JSX que especifica los atributos `value` y `onChange` para las etiquetas input.

Puedes extraer la lógica repetitiva dentro del Hook personalizado `useFormInput`:

**App.js**
```jsx
import { useFormInput } from './useFormInput.js';

export default function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');

  return (
    <>
      <label>
        Nombre:
        <input {...firstNameProps} />
      </label>
      <label>
        Apellido:
        <input {...lastNameProps} />
      </label>
      <p><b>Buenos días, {firstNameProps.value} {lastNameProps.value}.</b></p>
    </>
  );
}
```

**useFormInput.js**
```jsx
import { useState } from 'react';

export function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  function handleChange(e) {
    setValue(e.target.value);
  }

  const inputProps = {
    value: value,
    onChange: handleChange
  };

  return inputProps;
}
```

Muestra:

![[8-reutilizar-la-logica-con-custom-hooks-4.png]]


**Ten en cuenta que solo se declara _una_ variable de estado denominada `value**`.

**Sin embargo, el componente `Form` invoca a `useFormInput` _dos veces**_:

```jsx
function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');
  // ...
```

¡Es por eso que funciona como declarar dos variables de estado separadas!

**Los Hooks personalizados permiten compartir la _lógica con estado_ pero no _el estado en sí._ Cada invocación al Hook es completamente independiente de cualquier otra invocación al mismo Hook.** Es por eso que los dos sandboxes anteriores son completamente equivalentes. Si lo deseas, desplázate hacia atrás y compáralos. El comportamiento antes y después de extraer un Hook personalizado es idéntico.

Cuando necesites compartir el estado en sí entre múltiples componentes, [levántalo y pásalo hacia abajo](https://es.react.dev/learn/sharing-state-between-components).

## ⭐ Paso de valores reactivos entre Hooks 

**El código dentro de los Hooks personalizados se volverá a ejecutar durante cada nuevo renderizado de tu componente**. Por eso, como los componentes, los Hooks personalizados [deben ser puros.](https://es.react.dev/learn/keeping-components-pure) ¡Piensa en el código de los Hooks personalizados como parte del cuerpo de tu componente!

**Dado que los Hooks personalizados se rerenderizan en conjunto con tu componente, siempre reciben las últimas props y estado**. Para ver qué significa esto, considera este ejemplo de una sala de chat. Cambia la URL del servidor o la sala de chat seleccionada:

**App.js**
```jsx
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Elige la sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="viaje">viaje</option>
          <option value="música">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
      />
    </>
  );
}
```

**ChatRoom.js**
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';
import { showNotification } from './notifications.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.on('message', (msg) => {
      showNotification('Nuevo mensaje: ' + msg);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        URL del servidor:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>¡Bienvenido a la sala {roomId}!</h1>
    </>
  );
}
```

**chat.js**
```jsx
export function createConnection({ serverUrl, roomId }) {
  // Una implementación real en realidad se conectaría al servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Se esperaba que serverUrl fuera un string: Recibió: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Se esperaba que el roomId fuera un string. Recibió: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ Conectando a la sala "' + roomId + '" en ' + serverUrl + '...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hola')
          } else {
            messageCallback('xD');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ Desconectado de la sala "' + roomId + '" en ' + serverUrl + '');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('No se puede agregar la función controladora dos veces.');
      }
      if (event !== 'message') {
        throw Error('Solo se admite el evento "message".');
      }
      messageCallback = callback;
    },
  };
}
```

**notifications.js**
```jsx
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

Muestra:

![[8-reutilizar-la-logica-con-custom-hooks-5.png]]

**Cuando cambias `serverUrl` o `roomId`, el Efecto [«reacciona» a tus cambios](https://es.react.dev/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) y se vuelve a sincronizar**. Puedes saber por los mensajes de la consola que el chat se vuelve a conectar cada vez que cambian las dependencias del Efecto.

Ahora mueve el código del Efecto dentro de un Hook personalizado:

```jsx
export function useChatRoom({ serverUrl, roomId }) { // 👈
  useEffect(() => { // 👈
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('Nuevo mensaje: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // 👈
}
```

**Esto permite al componente `ChatRoom` invocar tu Hook personalizado sin preocuparse por cómo funciona internamente**:

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({ // 👈
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        URL del servidor:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>¡Bienvenido a la sala {roomId}!</h1>
    </>
  );
}
```

¡Esto parece mucho más simple! (Pero hace lo mismo).

Observa que la lógica _sigue respondiendo_ a los cambios de props y estado. Intenta editar la URL o la sala seleccionada:

**App.js**
```jsx
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Escoge la sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="viaje">viaje</option>
          <option value="música">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
      />
    </>
  );
}
```

**ChatRoom.js**
```jsx
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        URL del servidor:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>¡Bienvenido a la sala {roomId}!</h1>
    </>
  );
}
```

**useChatRoom.js**
```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';
import { showNotification } from './notifications.js';

export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('Nuevo mensaje: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

**chat.js**
```jsx
export function createConnection({ serverUrl, roomId }) {
  // Una implementación real en realidad se conectaría al servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Se esperaba que serverUrl fuera un string. Recibió: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Se esperaba que roomId fuera un string. Recibió: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ Conectando a la sala "' + roomId + '" en ' + serverUrl + '...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hola')
          } else {
            messageCallback('xD');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ Desconectado de la sala "' + roomId + '" en ' + serverUrl + '');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('No se puede agregar la función controladora dos veces.');
      }
      if (event !== 'message') {
        throw Error('Solo se admite el evento "message".');
      }
      messageCallback = callback;
    },
  };
}
```

**notifications.js**
```jsx
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

Observa cómo estás tomando el valor de devolución de un Hook:

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // 👈

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

y lo pasas como entrada a otro Hook:

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl // 👈
  });
  // ...
```

**Cada vez que tu componente `ChatRoom` se vuelve a renderizar, pasa el último `roomId` y `serverUrl` a tu Hook. Esta es la razón por la que tu Efecto se vuelve a conectar al chat cada vez que sus valores son diferentes después de volver a renderizarse**. (Si alguna vez trabajaste con un software de procesamiento de música, encadenar Hooks como este puede recordarte encadenar múltiples Efectos de audio, como agregar reverberación o coro. Es como si la salida de `useState` «alimentara» la entrada de `useChatRoom`).

### ⭐ Pasar controladores de eventos a los Hooks personalizados 

> [!danger]
>#### En construcción
>
>Esta sección describe una **API experimental que aún no se ha agregado a React,** por lo que aún no puedes usarla.

A medida que comiences a usar `useChatRoom` en más componentes, es posible que desees permitir diferentes componentes que personalicen su comportamiento. Por ejemplo, actualmente, la lógica de qué hacer cuando llega un mensaje está escrita dentro del Hook:

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => { // 👈
      showNotification('Nuevo mensaje: ' + msg); // 👈
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

Digamos que deseas mover esta lógica de regreso a tu componente:

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) { // 👈
      showNotification('Nuevo mensaje: ' + msg); // 👈
    }
  });
  // ...
```

Para hacer que esto funcione, cambia tu Hook para tomar `onReceiveMessage` como una de sus opciones nombradas:

```jsx
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) { // 👈
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onReceiveMessage(msg); // 👈
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl, onReceiveMessage]); // ✅ Todas las dependencias  👈declaradas
}
```

Esto funcionará, pero **hay una mejora que puedes hacer cuando tu Hook personalizado acepta controladores de eventos**.

**Agregar una dependencia en `onReceiveMessage` no es ideal, porque causará que el chat se vuelva a conectar cada vez que el componente se vuelva a renderizar**. [Envuelve este controlador en un Evento de Efecto para eliminarlo de las dependencias:](https://es.react.dev/learn/removing-effect-dependencies#wrapping-an-event-handler-from-the-props)

```jsx
import { useEffect, useEffectEvent } from 'react'; // 👈
// ...

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) { // 👈
  const onMessage = useEffectEvent(onReceiveMessage); // 👈

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg); // 👈
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ Todas las dependencias declaradas 👈
}
```

**Ahora el chat no se volverá a conectar cada vez que el componente `ChatRoom` se vuelva a renderizar**. Aquí hay una demostración completamente funcional de pasar un controlador de evento a un Hook personalizado con el que puedes manipular:

**App.js**
```jsx
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Elije la sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="viaje">viaje</option>
          <option value="música">música</option>
        </select>
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
      />
    </>
  );
}
```

**ChatRoom.js**
```jsx
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';
import { showNotification } from './notifications.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('Nuevo mensaje: ' + msg);
    }
  });

  return (
    <>
      <label>
        URL del servidor:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>¡Bienvenido a la sala {roomId}!</h1>
    </>
  );
}

```

**useChatRoom.js**
```jsx
import { useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection } from './chat.js';

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}

```

**chat.js**
```jsx
export function createConnection({ serverUrl, roomId }) {
  // Una implementación real en realidad se conectaría al servidor
  if (typeof serverUrl !== 'string') {
    throw Error('Se esperaba que serverUrl fuera un string. Recibió: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Se esperaba que roomId fuera un string. Recibió: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ Conectando a la sala "' + roomId + '" en ' + serverUrl + '...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hola')
          } else {
            messageCallback('xD');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ Desconectado de la sala "' + roomId + '" en ' + serverUrl + '');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('No se puede agregar la función controladora dos veces.');
      }
      if (event !== 'message') {
        throw Error('Solo se admite el evento "message".');
      }
      messageCallback = callback;
    },
  };
}
```

**notifications.js**
```jsx
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

Observa cómo ya no necesita saber _cómo_ `useChatRoom` funciona para usarlo. Podrías agregarlo a cualquier otro componente, pasar cualquier otra opción y funcionará de la misma manera. Ese es el poder de los Hooks personalizados.

## ⭐ ¿Cuándo usar Hooks personalizados? 

**No necesitas extraer un Hook personalizado por cada pequeño fragmento duplicado de tu código. Cierta duplicación está bien**. Por ejemplo, probablemente no sea necesario extraer un Hook `useFormInput` para envolver una sola invocación de `useState` como antes.

**==Sin embargo, cada vez que escribas un Efecto, considera si sería más claro envolverlo también en un Hook personalizado==**. [No deberías necesitar Efectos con mucha frecuencia,](https://es.react.dev/learn/you-might-not-need-an-effect) por lo que si estás escribiendo uno, significa que necesitas «salir de React» para sincronizar con algún sistema externo o hacer algo para lo que React no tiene API integrada. Envolver tu Efecto en un Hook personalizado permite comunicar con precisión tu intención y cómo fluyen los datos a través de este.

Por ejemplo, considera un componente `ShippingForm` que muestre dos menús desplegables: uno muestra la lista de ciudades y otro muestra la lista de áreas en la ciudad seleccionada. Puedes comenzar con un código que se vea así:

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  // Este Efecto recupera las ciudades de un país
  useEffect(() => { // 👈
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  // Este Efecto recupera las áreas para una ciudad seleccionada
  useEffect(() => { // 👈
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]);

  // ...
```

Aunque este código es bastante repetitivo, [es correcto mantener estos Efectos separados unos de otros.](https://es.react.dev/learn/removing-effect-dependencies#is-your-effect-doing-several-unrelated-things) Estos sincronizan dos cosas diferentes, por lo que no debes fusionarlos en un solo Efecto. En su lugar, **puedes simplificar el componente anterior `ShippingForm`, para extraer la lógica en común entre estos en tu propio Hook `useData**`:

```jsx
function useData(url) {
  const [data, setData] = useState(null); // 👈
  useEffect(() => { // 👈
    if (url) {
      let ignore = false;
      fetch(url)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setData(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [url]);
  return data;
}
```

Ahora puedes reemplazar ambos Efectos en los componentes `ShippingForm` con invocaciones a `useData`:

```jsx
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`); // 👈
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null); // 👈
  // ...
```

**Extraer un Hook personalizado hace que el flujo de datos sea explícito**. Alimentas la entrada `url` y obtienes la salida `data`. Al «ocultar» tu Efecto dentro de `useData`, también puedes evitar que alguien trabaje en el componente `ShippingForm` y agregue [dependencias innecesarias](https://es.react.dev/learn/removing-effect-dependencies). Idealmente, con el tiempo, la mayoría de los Efectos de tu aplicación estarán en Hooks personalizados.

#### Mantén tus Hooks personalizados enfocados en casos de uso concretos de alto nivel 

**Comienza por elegir el nombre de tu Hook personalizado. Si tiene dificultades para elegir un nombre claro, puede significar que tu Efecto está demasiado acoplado al resto de la lógica de tu componente y aún no está listo para ser extraído**.

Idealmente, el nombre de tu Hook personalizado debe ser lo suficientemente claro como para que incluso una persona que no escribe código con frecuencia, pueda tener una buena idea sobre lo que hace tu Hook personalizado, qué recibe y que devuelve:

- ✅ `useData(url)`
- ✅ `useImpressionLog(eventName, extraData)`
- ✅ `useChatRoom(options)`

Cuando sincronizas con un sistema externo, el nombre de tu Hook personalizado puede ser más técnico y usar un lenguaje específico para ese sistema. Es bueno siempre que sea claro para una persona familiarizada con ese sistema:

- ✅ `useMediaQuery(query)`
- ✅ `useSocket(url)`
- ✅ `useIntersectionObserver(ref, options)`

**Conserva tus Hooks personalizados enfocados en casos de uso específico de alto nivel. Evita crear y usar Hooks personalizados de «ciclo de vida» que actúen como alternativas y envoltorios convenientes para la propia API `useEffect**`.

- 🔴 `useMount(fn)`
- 🔴 `useEffectOnce(fn)`
- 🔴 `useUpdateEffect(fn)`

Por ejemplo, este Hook `useMount` intenta garantizar que parte del código solo se ejecute «en el montaje» del componente:

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // 🔴 Evita: usar Hooks personalizados de "ciclo de vida" 👈
  useMount(() => {
    const connection = createConnection({ roomId, serverUrl });
    connection.connect();

    post('/analytics/event', { eventName: 'visit_chat' });
  });
  // ...
}

// 🔴 Evita: usar Hooks personalizados de "ciclo de vida"
function useMount(fn) { // 👈
  useEffect(() => {
    fn();
  }, []); // 🔴 El Hook useEffect de React tiene una dependencia faltante: 'fn'
}
```

**Los Hooks personalizados de «ciclo de vida» como `useMount` no encajan bien en el paradigma de React.** Por ejemplo, este código de ejemplo tiene un error (no «reacciona» a los cambios de `roomId` o `serverUrl`), pero el linter no te avisará porque el linter solamente verifica las invocaciones directas del `useEffect`. No sabrá acerca de tu Hook.

Si estás escribiendo un Efecto, comienza por usar la API de React directamente:

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // ✅ Correcto: dos Efectos sin procesar separados por propósito

  useEffect(() => {
    const connection = createConnection({ serverUrl, roomId });
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]);

  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_chat', roomId });
  }, [roomId]);

  // ...
}
```

Luego, puedes (pero no tienes que hacerlo) extraer Hooks personalizados para diferentes casos de uso de alto nivel:

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // ✅ Genial: Hooks personalizados con el nombre de su propósito
  useChatRoom({ serverUrl, roomId });
  useImpressionLog('visit_chat', { roomId });
  // ...
}
```

**Un buen Hook personalizado hace que el código de invocación sea más declarativo al restringir lo que hace.** Por ejemplo, `useChatRoom(options)` solo puede conectarse a la sala de chat, mientras `useImpressionLog(eventName, extraData)` solo puede enviar un registro de impresión a las analíticas. Si la API de tu Hook personalizado no restringe los casos de uso y es muy abstracta, a la larga es probable que presente más problemas de los que resuelve.

### ⭐ Los Hooks personalizados te ayudan a migrar a mejores patrones 

Los Efectos son un [«escotilla de escape»](https://es.react.dev/learn/escape-hatches): los usas cuando necesitas «salir de React» y cuando no hay mejor solución integrada para tu caso de uso. Con el tiempo, el objetivo del equipo de React es reducir al mínimo la cantidad de Efectos en tu aplicación al proporcionar soluciones más específicas para problemas más específicos. Envolver los Efectos en Hooks personalizados facilita actualizar tu código cuando estas soluciones estén disponibles. Volvamos a este ejemplo:

Volvamos a este ejemplo:

**App.js**
```jsx
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Conectado' : '❌ Desconectado'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progreso guardado');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Guardar progreso' : 'Reconectando...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

**useOnlineStatus.js**
```jsx
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

En el ejemplo anterior, `useOnlineStatus` es implementado con un par de [`useState`](https://es.react.dev/reference/react/useState) y [`useEffect`.](https://es.react.dev/reference/react/useEffect) **Sin embargo, esta no es la mejor solución posible. Hay una serie de casos extremos que no considera. Por ejemplo, asume que cuando el componente se monta, `isOnline` ya es `true`, pero esto puede ser incorrecto si la red ya se desconectó**. Puede usar la API del navegador [`navigator.onLine`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine) para verificar eso, pero usarla directamente se rompería si ejecutas tu aplicación de React en un servidor para generar el HTML inicial. En resumen, este código podría mejorarse.

Afortunadamente, **==React 18 incluye una API llamada [`useSyncExternalStore`](https://es.react.dev/reference/react/useSyncExternalStore) que se encarga de todos estos problemas por ti. Así es como funciona tu Hook `useOnlineStatus`, reescrito para aprovechar esta nueva API==**:

**App.js**
```jsx
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Conectado' : '❌ Desconectado'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progreso guardado');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Guardar progreso' : 'Reconectando...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

**useOnlineStatus.js**
```jsx
import { useSyncExternalStore } from 'react';

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

export function useOnlineStatus() {
  return useSyncExternalStore( // 👈
    subscribe,
    () => navigator.onLine, // Cómo obtener el valor en el cliente 👈
    () => true // Cómo obtener el valor en el servidor 👈
  );
}

```


Observa cómo _no necesitabas cambiar ninguno de los componentes_ para realizar esta migración:

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus();
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  // ...
}
```

Esta es otra razón por la que envolver Efectos en Hooks personalizados suele ser beneficioso:

1. Haces que el flujo de datos hacia y desde tus Efectos sea muy explícito.
2. Permites que tus componentes se centren en la intención en lugar de la implementación exacta de tus Efectos.
3. Cuando React agrega nuevas funciones, puedes eliminar esos Efectos sin cambiar ninguno de tus componentes.

De manera similar a un [sistema de diseño,](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969) podría resultarte útil comenzar a extraer modismos comunes de los componentes de tu aplicación en Hooks personalizados. Esto mantendría el código de tus componentes enfocado en la intención y te permitirá evitar escribir Efectos sin procesar muy a menudo. También hay muchos Hooks personalizados excelentes mantenidos por la comunidad de React.

#### ⭐ Hook `use` - ¿Proporcionará React alguna solución integrada para la obtención de datos? 

Todavía estamos trabajando en los detalles, pero esperamos que en el futuro escribas la obtención de datos de esta manera:

```jsx
import { use } from 'react'; // ¡No disponible aún! // 👈

function ShippingForm({ country }) {
  const cities = use(fetch(`/api/cities?country=${country}`)); // 👈
  const [city, setCity] = useState(null);
  const areas = city ? use(fetch(`/api/areas?city=${city}`)) : null; // 👈
  // ...
```

Si usas Hooks personalizados como el anterior `useData` en tu aplicación, este requerirá menos cambios para migrar al enfoque recomendado eventualmente que si escribes Efectos sin procesar en cada componente manualmente. Sin embargo, el enfoque antiguo seguirá funcionando bien, entonces si te sientes feliz escribiendo Efectos sin procesar, puedes continuar haciéndolo.


### Hay más de una forma de hacerlo 

Supongamos que deseas implementar una animación de aparición gradual _desde cero_ usando la API del navegador [`requestAnimationFrame`](https://developer.mozilla.org/es/docs/Web/API/window/requestAnimationFrame). Podrías empezar con un Efecto que configure un bucle de animación. Durante cada fotograma de la animación, podrías cambiar la opacidad del nodo del DOM que [guardas en un ref](https://es.react.dev/learn/manipulating-the-dom-with-refs) hasta que alcance `1`. Tu código podría empezar así:

**App.js**
```jsx
import { useState, useEffect, useRef } from 'react';

function Welcome() {
  const ref = useRef(null);

  useEffect(() => {
    const duration = 1000;
    const node = ref.current;

    let startTime = performance.now();
    let frameId = null;

    function onFrame(now) {
      const timePassed = now - startTime;
      const progress = Math.min(timePassed / duration, 1);
      onProgress(progress);
      if (progress < 1) {
        // Todavía tenemos más fotogramas para pintar
        frameId = requestAnimationFrame(onFrame);
      }
    }

    function onProgress(progress) {
      node.style.opacity = progress;
    }

    function start() {
      onProgress(0);
      startTime = performance.now();
      frameId = requestAnimationFrame(onFrame);
    }

    function stop() {
      cancelAnimationFrame(frameId);
      startTime = null;
      frameId = null;
    }

    start();
    return () => stop();
  }, []);

  return (
    <h1 className="welcome" ref={ref}>
      Bienvenido
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remover' : 'Mostrar'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

Para que el componente sea más legible, podrías extraer la lógica en un Hook personalizado denominado `useFadeIn`.

**App.js**
```jsx
import { useState, useEffect, useRef } from 'react';
import { useFadeIn } from './useFadeIn.js';

function Welcome() {
  const ref = useRef(null);

  useFadeIn(ref, 1000);

  return (
    <h1 className="welcome" ref={ref}>
      Bienvenido
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remover' : 'Mostrar'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

**useFadeIn.js**
```jsx
import { useEffect } from 'react';

export function useFadeIn(ref, duration) {
  useEffect(() => {
    const node = ref.current;

    let startTime = performance.now();
    let frameId = null;

    function onFrame(now) {
      const timePassed = now - startTime;
      const progress = Math.min(timePassed / duration, 1);
      onProgress(progress);
      if (progress < 1) {
        // Aún tenemos más fotogramas que pintar
        frameId = requestAnimationFrame(onFrame);
      }
    }

    function onProgress(progress) {
      node.style.opacity = progress;
    }

    function start() {
      onProgress(0);
      startTime = performance.now();
      frameId = requestAnimationFrame(onFrame);
    }

    function stop() {
      cancelAnimationFrame(frameId);
      startTime = null;
      frameId = null;
    }

    start();
    return () => stop();
  }, [ref, duration]);
}
```

Podrías mantener el código de `useFadeIn` como está, pero también podrías refactorizarlo más. Por ejemplo, podrías extraer la lógica para configurar el bucle de animación fuera de `useFadeIn` en un nuevo Hook personalizado llamado `useAnimationLoop`:

**App.js**
```jsx
import { useState, useEffect, useRef } from 'react';
import { useFadeIn } from './useFadeIn.js';

function Welcome() {
  const ref = useRef(null);

  useFadeIn(ref, 1000);

  return (
    <h1 className="welcome" ref={ref}>
      Bienvenido
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remover' : 'Mostrar'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

**useFadeIn.js**
```jsx
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export function useFadeIn(ref, duration) {
  const [isRunning, setIsRunning] = useState(true);

  useAnimationLoop(isRunning, (timePassed) => {
    const progress = Math.min(timePassed / duration, 1);
    ref.current.style.opacity = progress;
    if (progress === 1) {
      setIsRunning(false);
    }
  });
}

function useAnimationLoop(isRunning, drawFrame) {
  const onFrame = useEffectEvent(drawFrame);

  useEffect(() => {
    if (!isRunning) {
      return;
    }

    const startTime = performance.now();
    let frameId = null;

    function tick(now) {
      const timePassed = now - startTime;
      onFrame(timePassed);
      frameId = requestAnimationFrame(tick);
    }

    tick();
    return () => cancelAnimationFrame(frameId);
  }, [isRunning]);
}
```


Sin embargo, no _tenías_ que hacer eso. Al igual que con las funciones normales, en última instancia, decides dónde dibujar los límites entre las diferentes partes de tu código. Por ejemplo, también podrías adoptar un enfoque muy diferente. En lugar de mantener la lógica en el Efecto, podrías mover la mayor parte de la lógica imperativa dentro de una [clase de JavaScript:](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Classes)

**useFadeIn.js**
```jsx
import { useState, useEffect } from 'react';
import { FadeInAnimation } from './animation.js';

export function useFadeIn(ref, duration) {
  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(duration);
    return () => {
      animation.stop();
    };
  }, [ref, duration]);
}

```


Los Efectos te permiten conectar React a sistemas externos. Cuanta más coordinación se necesita entre los Efectos (por ejemplo, para encadenar múltiples animaciones), más sentido tiene extraer completamente esa lógica de Efectos y Hooks _completamente_ como en el sandbox anterior. Luego, el código que extraes _se convierte_ en el «sistema externo». Esto permite que tus Efectos se mantengan simples porque solo necesitan enviar mensajes al sistema que moviste fuera de React.

Los ejemplos anteriores asumen que la lógica de aparición gradual debe escribirse en JavaScript. Sin embargo, esta particular animación de aparición gradual es más simple y eficiente de implementar con una simple [animación CSS:](https://developer.mozilla.org/es/docs/Web/CSS/CSS_Animations/Using_CSS_animations)

**App.js**
```jsx
import { useState, useEffect, useRef } from 'react';
import './welcome.css';

function Welcome() {
  return (
    <h1 className="welcome">
      Bienvenido
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remover' : 'Mostrar'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

**welcome.css**
```css
.welcome {
  color: white;
  padding: 50px;
  text-align: center;
  font-size: 50px;
  background-image: radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%);

  animation: fadeIn 1000ms;
}

@keyframes fadeIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}

```

¡A veces ni siquiera necesitas un Hook!

## Recapitulación

- Los Hooks personalizados te permiten compartir lógica entre componentes.
- Los Hooks personalizados deben nombrarse comenzando con `use`, seguido por una letra mayúscula.
- Los Hooks personalizados solo comparten lógica con estado, no el estado en sí.
- Puedes pasar valores reactivos de un Hook a otro, y se mantienen actualizados.
- Todos los Hooks se vuelven a ejecutar cada vez que tu componente se vuelve a renderizar.
- El código de tus Hooks debe ser puro, como el código de tu componente.
- Envuelve controladores de eventos recibidos por Hooks personalizados en Eventos de Efecto.
- No hagas Hooks personalizados como `useMount`. Conserva su propósito específico.
- Depende de ti el cómo y dónde elegir los límites de tu código.