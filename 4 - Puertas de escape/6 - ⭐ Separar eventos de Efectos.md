Los controladores de eventos solo se vuelven a ejecutar cuando vuelves a realizar la misma interacción. A diferencia de los controladores de eventos, los Efectos se resincronizan si algún valor que leen, como una prop o una variable de estado, es diferente de lo que era durante el último renderizado. **A veces, también quieres una mezcla de ambos comportamientos: un Efecto que se vuelve a ejecutar en respuesta a algunos valores pero no a otros. Esta página te enseñará cómo hacerlo**.

### Aprenderás

- Cómo escoger entre un controlador de evento y un Efecto
- Por qué los Efectos son reactivos, y los controladores de eventos no lo son
- Qué hacer cuando quieres que una parte del código de tu Efecto no sea reactivo
- Qué son los eventos de Efecto y cómo extraerlos de tus Efectos
- Cómo leer las últimas props y estados de los Efectos usando Eventos de Efecto

## Elegir entre controladores de eventos y Efectos 

Primero, vamos a recapitular la diferencia entre controladores de eventos y Efectos.

Imagina que estas implementando un componente de sala de chat. Tus requerimientos se verán así:

1. Tu componente debería conectarse de forma automática a la sala de chat seleccionada.
2. Cuándo hagas click al botón «Enviar», debería enviar un mensaje al chat.

Digamos que ya tienes el código implementado para ello, pero no estas seguro de donde ponerlo. ¿Deberías de usar controladores de eventos o Efectos? Cada vez que necesites contestar este pregunta, considera [_por qué_ se necesita ejecutar el código.](https://es.react.dev/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)

### ⭐ Los controladores de eventos se ejecutan en respuesta a interacciones especificas

**Desde la perspectiva del usuario, el envío de un mensaje debe producirse _porque_ se hace clic en particular en el botón «Enviar»**. El usuario se enfadará bastante si envías su mensaje en cualquier otro momento o por cualquier otro motivo. Esta es la razón por la que enviar un mensaje debería ser un controlador de evento. Los controladores de eventos te permiten controlar interacciones específicas:

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');
  // ...
  function handleSendClick() { // 👈
    sendMessage(message); // 👈
  }
  // ...
  return (
    <>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Enviar</button>
    </>
  );
}
```

**Con un controlador de Evento, puedes estar seguro que `sendMessage(message)` _únicamente_ se activará si el usuario presiona el botón**.

### ⭐ Los Efectos se ejecutan siempre que es necesaria la sincronización 

Recuerda que también necesitas mantener el componente conectado a la sala de chat. ¿Dónde va ese código?

**La _razón_ para ejecutar este código no es ninguna interacción en particular**. No es importante, el cómo o de qué forma el usuario navegó hasta la sala de chat. Ahora que ellos están viéndola y pueden interactuar con ella, el componente necesita mantenerse conectado al servidor de chat seleccionado. **Incluso si el componente de la sala de chat fuera la pantalla inicial de tu aplicación y el usuario no ha realizado ningún tipo de interacción, _todavía_ necesitarías conectarte. Es por eso que es un Efecto**:

```jsx
function ChatRoom({ roomId }) {
  // ...
  useEffect(() => { // 👈
    const connection = createConnection(serverUrl, roomId); // 👈
    connection.connect(); // 👈
    return () => { // 👈
      connection.disconnect(); // 👈
    };
  }, [roomId]); // 👈
  // ...
}
```

Con este código, puedes estar seguro que siempre hay una conexión activa al servidor de chat seleccionado actualmente, _independientemente_ de las interacciones específicas realizadas por el usuario. **Si el usuario solo ha abierto tu aplicación, seleccionado una sala diferente o navegado a otra pantalla y volvió, tu Efecto garantiza que el componente _permanecerá sincronizado_ con la sala seleccionada a actualmente, y [volverá a conectarse cuando sea necesario.](https://es.react.dev/learn/lifecycle-of-reactive-effects#why-synchronization-may-need-to-happen-more-than-once)**

**App.js**
```jsx
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  function handleSendClick() {
    sendMessage(message);
  }

  return (
    <>
      <h1>¡Bienvenido a la sala {roomId}!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Enviar</button>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Selecciona la sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="viaje">viaje</option>
          <option value="música">música</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Cerrar chat' : 'Abrir chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

**chat.js**
```jsx
export function sendMessage(message) {
  console.log('🔵 Enviaste: ' + message);
}

export function createConnection(serverUrl, roomId) {
  // Una aplicación real se conectaría al servidor
  return {
    connect() {
      console.log('✅ Conectando a la sala "' + roomId + '" en ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectando de la sala "' + roomId + '" en ' + serverUrl);
    }
  };
}
```

Muestra:

![[5-ciclo-de-vida-de-los-efectos-reactivos-1.png]]

Al presionar sobre el botón, el componente del chat se montará y se mostrarán 3 mensajes en la consola:

![[6-separar-eventos-de-efectos-1.png]]

## 🌟 Valores reactivos y lógica reactiva 

**Intuitivamente, podría decirse que los controladores de eventos siempre se activan «manualmente», por ejemplo, al pulsar un botón. Los Efectos, en cambio, son «automáticos»: se ejecutan y se vuelven a ejecutar tantas veces como sea necesario para mantenerse sincronizados**.

**Hay una forma más precisa de pensar en esto**.

Las propiedades, estados, y variables declarados dentro del cuerpo de tu componente son llamados valores reactivos. En este ejemplo, `serverUrl` no es un valor reactivo, pero `roomId` y `message` sí lo son. Participan en el flujo de datos de renderizado:

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ...
}
```

**Valores reactivos como estos pueden cambiar debido a un re-renderizado**. Por ejemplo, el usuario puede editar el `message` o elegir un `roomId` diferente en un desplegable. **Los controladores de eventos y Efectos responden a los cambios de manera diferente**:

- **La lógica dentro de los controladores de eventos no es _reactiva._** **No se ejecutará de nuevo a menos que el usuario vuelva a realizar la misma interacción (por ejemplo, un clic)**. Los controladores de eventos pueden leer valores reactivos sin «reaccionar» a sus cambios.
- **La lógica dentro de los Efectos es _reactiva._** **Si tu Efecto lee un valor reactivo, [tienes que especificarlo como una dependencia](https://es.react.dev/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values)** Luego, si una nueva renderización hace que ese valor cambie, React volverá a ejecutar la lógica de tu Efecto con el nuevo valor.

Volvamos al ejemplo anterior para ilustrar esta diferencia.

### La lógica dentro de los controladores de eventos no es reactiva

Echa un vistazo a esta línea de código. ¿Esta lógica debería ser reactiva o no?

```jsx
    // ...
    sendMessage(message);
    // ...
```

Desde la perspectiva del usuario, **un cambio en el `message` _no_ significa que quiera enviar un mensaje.** Solo significa que el usuario está escribiendo. En otras palabras, **la lógica que envía un mensaje no debería ser reactiva. No debería volver a ejecutarse solo porque el valor reactivo ha cambiado. Por eso pertenece al controlador de evento**:

```jsx
  function handleSendClick() {
    sendMessage(message);
  }
```

Los controladores de eventos no son reactivos, por lo que `sendMessage(message)` solo se ejecutará cuando el usuario pulse el botón Enviar.

### La lógica dentro de los Efectos es reactiva

Ahora volvamos a estas líneas:

```jsx
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ...
```

Desde la perspectiva del usuario, **un cambio en el `roomId` _significa_ que quieren conectarse a una sala diferente.** En otras palabras, **la lógica para conectarse a la sala debe ser reactiva**. Usted _quiere_ estas líneas de código para «mantenerse al día» con el valor reactivo, y para ejecutar de nuevo si ese valor es diferente. **Es por eso que pertenece en un Efecto**:

```jsx
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 👈
    connection.connect(); // 👈
    return () => {
      connection.disconnect()
    };
  }, [roomId]);
```

Los Efectos son reactivos, por lo que `createConnection(serverUrl, roomId)` y `connection.connect()` se ejecutarán para cada valor distinto de `roomId`. Tu Efecto mantiene la conexión de chat sincronizada con la sala seleccionada en ese momento.

## ⭐ Extraer lógica no reactiva fuera de los Efectos

**Las cosas se vuelven más complicadas cuando tu quieres combinar lógica reactiva con lógica no reactiva**.

Por ejemplo, imagina que quieres mostrar una notificación cuando el usuario se conecta al chat. Lees el tema actual (oscuro o claro) de los accesorios para poder mostrar la notificación en el color correcto:

```jsx
function ChatRoom({ roomId, theme }) { // 👈
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => { // 👈
      showNotification('¡Conectado!', theme); // 👈
    });
    connection.connect();
    // ...
```

Sin embargo, `theme` es un valor reactivo (puede cambiar como resultado del re-renderizado), y [cada valor reactivo leído por un Efecto debe ser declarado como su dependencia](https://es.react.dev/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) Ahora tienes que especificar `theme` como una dependencia de tu Efecto:

```jsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('¡Conectado!', theme); // 👈
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]); // ✅ Todas las dependencias declaradas 👈
  // ...
```

Juegue con este ejemplo y vea si puede detectar el problema con esta experiencia de usuario:

**App.js**
```jsx
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('¡Conectado!', theme);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]);

  return <h1>¡Bienvenido a la sala {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Escoje la sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="viaje">viaje</option>
          <option value="música">música</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema oscuro
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

**chat.js**
```jsx
export function createConnection(serverUrl, roomId) {
  // Una aplicación real se conectaría al servidor
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('No se puede agregar el controlador dos veces.');
      }
      if (event !== 'connected') {
        throw Error('Solo se admite el evento "connected".');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

**notifications.js**
```jsx
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
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

Muestra la notificación a penas se monta el componente o cuando se cambia de chat:

![[6-separar-eventos-de-efectos-2.png]]

Pero cuando cambiamos de tema también se muestra la notificación (cosa que no debería):

![[6-separar-eventos-de-efectos-3.png]]

**Cuando el `roomId` cambia, el chat se reconecta como es de esperar. Pero como `theme` también es una dependencia, el chat _también_ se reconecta cada vez que cambias entre el tema oscuro y el claro. Esto no es bueno**.

En otras palabras, _**no_ quieres que esta línea sea reactiva, aunque esté dentro de un Efecto (que es reactivo)**:

```jsx
      // ...
      showNotification('¡Conectado!', theme);
      // ...
```

Necesitas una forma de separar esta lógica no reactiva del Efecto reactivo que la rodea.

### 🌟 `useEffectEvent` - Declaración de un Evento de Efecto

> [!danger]
> #### En construcción
> Esta sección describe una API **experimental que aún no se ha publicado** en una versión estable de React.

Utiliza un Hook especial llamado [`useEffectEvent`](https://es.react.dev/reference/react/experimental_useEffectEvent) para extraer esta lógica no reactiva de su Efecto:

```jsx
import { useEffect, useEffectEvent } from 'react'; // 👈

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => { // 👈
    showNotification('¡Conectado!', theme); // 👈
  });
  // ...
```

**Aquí, `onConnected` se llama un _Evento de Efecto._ Es una parte de tu lógica de Efecto, pero se comporta mucho más como un controlador de evento**. La lógica dentro de él no es reactiva, y siempre «ve» los últimos valores de tus props y estado.

**Ahora puedes llamar al Evento de Efecto `onConnected` desde dentro de tu Efecto**:

```jsx
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => { // 👈
    showNotification('¡Conectado!', theme); // 👈
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected(); // 👈
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas las dependencias declaradas 👈
  // ...
```

**Esto resuelve el problema**. Ten en cuenta que has tenido que _eliminar_ `onConnected` de la lista de dependencias de tu Efecto. **Los Eventos de Efecto no son reactivos y deben ser omitidos de las dependencias.** Verifica que el nuevo comportamiento funciona como esperas:

**App.js**
```jsx
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => { // 👈
    showNotification('¡Conectado!', theme); // 👈
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected(); // 👈
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Todas las dependencias declaradas 👈

  return <h1>¡Bienvenido a la sala {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Escoje la sala de chat:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="viaje">viaje</option>
          <option value="música">música</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Usar tema oscuro
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

**chat.js**
```jsx
export function createConnection(serverUrl, roomId) {
  // Una aplicación real se conectaría al servidor
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('No se puede añadir el controlador dos veces.');
      }
      if (event !== 'connected') {
        throw Error('Solo se admite el evento "connected".');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

Muestra la notificación a penas se monta el componente o cuando se cambia de chat:

![[6-separar-eventos-de-efectos-2.png]]

Pero ahora cuando cambiamos de tema, ya no se dispara la notificación:

![[6-separar-eventos-de-efectos-4.png]]

**Puedes pensar que los Eventos de Efecto son muy similares a los controladores de eventos. La principal diferencia es que los controladores de eventos se ejecutan en respuesta a las interacciones del usuario, mientras que los Eventos de Efecto son disparados por ti desde los Efectos**. Los Eventos de Efecto te permiten «romper la cadena» entre la reactividad de los Efectos y el código que no debería ser reactivo.

### ⭐ Leer las últimas propiedades y el estado con los Eventos de Efecto

> [!danger]
> #### En construcción
> Esta sección describe una API **experimental que aún no se ha publicado** en una versión estable de React.

**Los Eventos de Efecto le permiten arreglar muchos patrones en los que podría verse tentado a eliminar el linter de dependencias**.

Por ejemplo, digamos que tienes un Efecto para registrar las visitas a la página:

```jsx
function Page() {
  useEffect(() => {
    logVisit();
  }, []);
  // ...
}
```

Más tarde, añades múltiples rutas a tu sitio. Ahora tu componente `Page` recibe una propiedad `url` con la ruta actual. Quieres pasar la `url` como parte de tu llamada `logVisit`, pero el linter de dependencias se queja:

```jsx
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, []); // 🔴 Hook de React useEffect tiene una dependencia que falta: 'url' ❌
  // ...
}
```

**Piense en lo que quiere que haga el código**. Usted _quiere_ registrar una visita separada para diferentes URLs ya que cada URL representa una página diferente. En otras palabras, **esta llamada a `logVisit` _debería_ ser reactiva con respecto a la `url`. Por eso, en este caso, tiene sentido seguir el linter de dependencias, y añadir `url` como dependencia**:

```jsx
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, [url]); // ✅ Todas las dependencias declaradas
  // ...
}
```

Supongamos ahora que desea incluir el número de artículos en el carrito de compras junto con cada visita a la página:

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext); // 👈
  const numberOfItems = items.length; // 👈

  useEffect(() => {
    logVisit(url, numberOfItems); // 👈
  }, [url]); // 🔴 React Hook useEffect has a missing dependency: 'numberOfItems'
  // ...
}
```

**Has utilizado `numberOfItems` dentro del Efecto, por lo que el linter te pide que lo añadas como dependencia. Sin embargo, _no_ quieres que la llamada a `logVisit` sea reactiva con respecto a `numberOfItems`**. Si el usuario pone algo en el carro de la compra, y el `numberOfItems` cambia, esto _no significa_ que el usuario haya visitado la página de nuevo. En otras palabras, _**visitar la página_ es, en cierto sentido, un «evento». Ocurre en un momento preciso**.

**Divide el código en dos partes**:

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent(visitedUrl => { // 👈
    logVisit(visitedUrl, numberOfItems); // 👈
  });

  useEffect(() => {
    onVisit(url); // 👈
  }, [url]); // ✅ Todas las dependencias declaradas
  // ...
}
```

**Aquí, `onVisit` es un Evento de Efecto. El código que contiene no es reactivo. Por eso puedes usar `numberOfItems` (¡o cualquier otro valor reactivo!) sin preocuparte de que cause que el código circundante se vuelva a ejecutar con los cambios**.

**Por otro lado, el Efecto en sí sigue siendo reactivo**. El código dentro del Efecto utiliza la propiedad `url`, por lo que el Efecto se volverá a ejecutar después de cada rerenderizado con una `url` diferente. Esto, a su vez, llamará al Evento de Efecto «`onVisit`».

Como resultado, se llamará a `logVisit` por cada cambio en la `url`, y siempre se leerá el último `numberOfItems`. Sin embargo, si `numberOfItems` cambia por sí mismo, esto no hará que se vuelva a ejecutar el código.

> [!info]
Puede que te preguntes si podrías llamar a `onVisit()` sin argumentos, y leer la `url` que contiene:
>
>```jsx
  >const onVisit = useEffectEvent(() => {
    >logVisit(url, numberOfItems); // 👈
  >});
>
  >useEffect(() => {
    >onVisit(); // 👈
  >}, [url]);
>```
>
Esto funcionaría, pero es mejor pasar esta `url` al Evento de Efecto explícitamente. **Al pasar `url` como argumento a tu Evento de Efecto, estás diciendo que visitar una página con una `url` diferente constituye un «evento» separado desde la perspectiva del usuario.** La `visitedUrl` es una parte del «evento» que ocurrió:
>
>```jsx
  >const onVisit = useEffectEvent(visitedUrl => { // 👈
    >logVisit(visitedUrl, numberOfItems); // 👈
  >});
>
  >useEffect(() => {
    >onVisit(url); // 👈
  >}, [url]);
>```
>
Desde que tu Evento de Efecto «pregunta» explícitamente por la `visitedUrl`, ahora no puedes eliminar accidentalmente `url` de las dependencias del Efecto. Si eliminas la dependencia `url` (provocando que distintas visitas a la página se cuenten como una), el linter te advertirá de ello. Quieres que `onVisit` sea reactivo con respecto a la `url`, así que en lugar de leer la `url` dentro (donde no sería reactivo), la pasas _desde_ tu Efecto.
>
>Esto es especialmente importante si hay alguna lógica asíncrona dentro del Efecto:
>
>```jsx
  >const onVisit = useEffectEvent(visitedUrl => {
    >logVisit(visitedUrl, numberOfItems);
  >});
>
  >useEffect(() => {
    >setTimeout(() => {
      >onVisit(url);
    >}, 5000); // Retraso en el registro de visitas
  >}, [url]);
>```
>
>Aquí, `url` dentro de `onVisit` corresponde a la _última_ `url` (que podría haber cambiado), pero `visitedUrl` corresponde a la `url` que originalmente causó que este Efecto (y esta llamada a `onVisit`) se ejecutara.

#### ¿Está bien suprimir el linter de dependencia en su lugar? 

En las bases de código existentes, a veces puede ver la regla lint suprimida de esta manera:

```jsx
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
    // 🔴 Evite suprimir el linter de este modo: 👈
    // eslint-disable-next-line react-hooks/exhaustive-deps 👈
  }, [url]);
  // ...
}
```

**==Después de que `useEffectEvent` se convierta en una parte estable de React, recomendamos nunca suprimir el linter==**.

La primera desventaja de suprimir la regla es que React ya no te avisará cuando tu Efecto necesite «reaccionar» a una nueva dependencia reactiva que hayas introducido en tu código. En el ejemplo anterior, añadiste `url` a las dependencias _porque_ React te lo recordó. Si desactivas el linter, ya no recibirás esos recordatorios para futuras ediciones de ese Efecto. Esto conduce a errores.

Aquí hay un ejemplo de un error confuso causado por la supresión del linter. En este ejemplo, se supone que la función `handleMove` lee el valor actual de la variable de estado `canMove` para decidir si el punto debe seguir al cursor. Sin embargo, `canMove` es siempre `true` dentro de `handleMove`.

¿Puedes ver por qué?

**App.js**
```jsx
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  function handleMove(e) {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  }

  useEffect(() => {
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        El punto puede moverse
      </label>
      <hr />
      <div style={{
        position: 'absolute',
        backgroundColor: 'pink',
        borderRadius: '50%',
        opacity: 0.6,
        transform: `translate(${position.x}px, ${position.y}px)`,
        pointerEvents: 'none',
        left: -20,
        top: -20,
        width: 40,
        height: 40,
      }} />
    </>
  );
}
```

El punto se mueve en el plano aun cuando se marca la casilla para que no lo haga:

![[6-separar-eventos-de-efectos-5.png|670]]

**El problema con este código está en suprimir el linter de dependencia. Si eliminas la supresión, verás que este Efecto debería depender de la función `handleMove`. Esto tiene sentido**: `handleMove` se declara dentro del cuerpo del componente, lo que lo convierte en un valor reactivo. Cada valor reactivo debe ser especificado como una dependencia, ¡o puede potencialmente volverse obsoleto con el tiempo!

El autor del código original ha «mentido» a React diciendo que el Efecto no depende (`[]`) de ningún valor reactivo. Por eso React no ha resincronizado el Efecto después de que `canMove` haya cambiado (y `handleMove` con él). Debido a que React no ha resincronizado el Efecto, el `handleMove` adjunto como listener es la función `handleMove` creada durante el render inicial. Durante el render inicial, `canMove` era `true`, por lo que `handleMove` del render inicial verá siempre ese valor.

**Si nunca suprimes el linter, nunca verás problemas con valores obsoletos.**

Con `useEffectEvent`, no hay necesidad de «mentir» al linter, y el código funciona como cabría esperar:

**App.js**
```jsx
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  const onMove = useEffectEvent(e => {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  });

  useEffect(() => {
    window.addEventListener('pointermove', onMove);
    return () => window.removeEventListener('pointermove', onMove);
  }, []);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        El punto puede moverse
      </label>
      <hr />
      <div style={{
        position: 'absolute',
        backgroundColor: 'pink',
        borderRadius: '50%',
        opacity: 0.6,
        transform: `translate(${position.x}px, ${position.y}px)`,
        pointerEvents: 'none',
        left: -20,
        top: -20,
        width: 40,
        height: 40,
      }} />
    </>
  );
}
```

Ahora sí se activamos en la casilla el punto rojo deja de moverse cuando marcamos la casilla:

![[6-separar-eventos-de-efectos-6.png]]

**Esto no significa que `useEffectEvent` sea _siempre_ la solución correcta. Solo deberías aplicarlo a las líneas de código que no quieres que sean reactivas**. En el sandbox anterior, no querías que el código del Efecto fuera reactivo con respecto a `canMove`. Por eso tenía sentido extraer un Evento de Efecto.

Leer [Eliminar dependencias de Efectos](https://es.react.dev/learn/removing-effect-dependencies) para otras alternativas correctas a la supresión del linter.

### Limitaciones de los Eventos de Efecto 

> [!danger]
> #### En construcción
> Esta sección describe una API **experimental que aún no se ha publicado** en una versión estable de React.

**==Los Eventos de Efecto tienen un uso muy limitado==**:

- **Llámalos solo desde dentro Efectos.**
- **Nunca los pases a otros componentes o Hooks.**

Por ejemplo, no declares y pases un Evento de Efecto así:

```jsx
function Timer() {
  const [count, setCount] = useState(0);

  const onTick = useEffectEvent(() => { // 👈
    setCount(count + 1); // 👈
  });

  useTimer(onTick, 1000); // 🔴 Evitar: Pasar Eventos de Efecto 👈

  return <h1>{count}</h1>
}

function useTimer(callback, delay) {
  useEffect(() => {
    const id = setInterval(() => {
      callback();
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay, callback]); // Necesitas especificar "callback" en las dependencias
}
```

En su lugar, declare siempre los Eventos de Efecto directamente junto a los Efectos que los utilizan:

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  useTimer(() => {
    setCount(count + 1);
  }, 1000);
  return <h1>{count}</h1>
}

function useTimer(callback, delay) {
  const onTick = useEffectEvent(() => { // 👈
    callback(); // 👈
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick(); // ✅ Bien: Solo se activa localmente dentro de un Efecto 👈
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay]); // No es necesario especificar "onTick" (un evento de Efecto) como dependencia. 👈
}
```

Los Eventos de Efecto son «piezas» no reactivas de tu código de Efecto. Deben estar junto al Efecto que los utiliza.

## Recapitulación

- Los controladores de eventos se ejecutan en respuesta a interacciones específicas.
- Los Efectos se ejecutan siempre que es necesaria la sincronización.
- La lógica dentro de los controladores de eventos no es reactiva.
- La lógica dentro de Efectos es reactiva.
- Puede mover la lógica no reactiva de Efectos a Eventos de Efecto.
- Llame a Eventos de Efecto solo desde dentro de Efectos.
- No pase Eventos de Efecto a otros componentes o Hooks.