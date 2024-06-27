Los Efectos tienen un diferente ciclo de vida al de los componentes. Los componentes pueden montarse, actualizarse o desmontarse. Un Efecto solo puede hacer dos cosas: empezar a sincronizar algo y luego dejar de sincronizarlo. Este ciclo puede suceder varias veces si tu Efecto depende de props y estado que cambian con el tiempo. React provee una regla del _linter_ para comprobar que hayas especificado las dependencias de tu Efecto correctamente. Esto mantiene tu Efecto sincronizado con las últimas props y estado.

### Aprenderás

- Cómo un ciclo de vida de un Efecto es diferente del ciclo de vida de un componente
- Cómo pensar en cada Efecto de forma aislada
- Cuándo tu Efecto necesita volver a sincronizarse, y por qué
- Cómo se determinan las dependencias de tu Efecto
- Qué significa para un valor ser reactivo
- Qué significa un _array_ de dependencias vacío
- Cómo React verifica con un _linter_ que tus dependencias son correctas
- Qué hacer cuanto no estás de acuerdo con el _linter_

## El ciclo de vida de un Efecto 

Cada componente de React pasa por el mismo ciclo de vida:

- Un componente se _monta_ cuando es agregado a la pantalla.
- Un componente se _actualiza_ cuando recibe nuevas props o estado, por lo general en respuesta de una interacción.
- Un componente se _desmonta_ cuando es removido de la pantalla.

**Es una buena manera de pensar sobre los componentes, pero _no_ sobre los Efectos.** En cambio, intenta pensar en cada Efecto independientemente del ciclo de vida de tu componente. Un Efecto describe cómo [sincronizar un sistema externo](https://es.react.dev/learn/synchronizing-with-effects) con las props actuales y el estado. A medida que tu código cambia, la sincronización tendrá que suceder más o menos a menudo.

Para ilustrar este punto, considera este Efecto que conecta tu componente a un servidor de chat:

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

El cuerpo de tu Efecto especifica cómo **iniciar la sincronización:**

```jsx
    // ...
    const connection = createConnection(serverUrl, roomId); // 👈
    connection.connect(); // 👈
    return () => {
      connection.disconnect();
    };
    // ...
```

La función de limpieza devuelta por tu Efecto especifica cómo **detener la sincronización:**

```jsx
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect(); // 👈
    };
    // ... ...
```

Intuitivamente, podrías pensar que React empezaría a sincronizarse cuando el componente se monta y dejaría de sincronizarse cuando el componente se desmonta. Sin embargo, ¡Este no es el final de la historia! A veces, también puede ser necesario **iniciar y detener la sincronización varias veces** mientras el componente permanece montado.

Veamos _por qué_ esto es necesario, _cuándo_ sucede, y _cómo_ se puede controlar este comportamiento.

> [!note]
>Algunos Efectos no devuelven una función de limpieza en absoluto. [Más a menudo que no,](https://es.react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) tu querrás devolver uno, pero si no, React se comportará como si devolviera una función de limpieza vacía.

### Por qué la sincronización puede necesitar suceder mas de una vez 

Imagina que este componente `ChatRoom` recibe una prop `roomId` que el usuario selecciona de un menú desplegable. Digamos que inicialmente el usuario selecciona la sala `"general"` como el `roomId`. Tu aplicación muestra la sala de chat `"general"`:

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) { // 👈
  // ...
  return <h1>¡Bienvenido a la sala {roomId}!</h1>;
}
```

Después que se muestre la UI, React ejecutará el Efecto para **iniciar la sincronización.** Se conecta a la sala `"general"`:

```jsx
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Conecta a la sala "general" 👈
    connection.connect(); // 👈
    return () => {
      connection.disconnect(); // Desconecta de la sala "general"
    };
  }, [roomId]);
  // ...
```

Hasta ahora, todo bien.

Luego, el usuario selecciona una sala diferente en el menú desplegable (por ejemplo, `"viaje"`). Primero, React actualizará la UI:

```jsx
function ChatRoom({ roomId /* "viaje" */ }) { // 👈
  // ...
  return <h1>¡Bienvenido a la sala {roomId}!</h1>;
}
```

Piensa en que debería suceder luego. El usuario ve que `"viaje"` es la sala de chat en la UI. Sin embargo, el Efecto que se ejecutó la última vez aun está conectada a la sala `"general"`. **La prop `"roomId"` ha cambiado, asi que lo que el Efecto hizo en ese entonces (conectarse a la sala `"general"`) ya no coincide con la UI.**

En este punto, deseas que React haga dos cosas:

1. Que detenga la sincronización con el antiguo `roomId` (desconectarse de la sala `"general"`)
2. Que inicie la sincronización con el nuevo `roomId` (conectarse a la sala `"viaje"`)

**Afortunadamente, ¡ya has enseñado a React a cómo hacer ambas cosas!** El cuerpo del Efecto especifica cómo iniciar la sincronización, y su función de limpieza especifica cómo detener la sincronización. Todo lo que React necesita hacer ahora es llamarlos en el orden correcto y con las props y estado correctos. Veamos cómo sucede esto exactamente.

### Cómo React vuelve a sincronizar tu Efecto

Recuerda que tu componente `ChatRoom` había recibido un nuevo valor para su prop `roomId`. Solía ser `"general"`, y ahora es `"viaje"`. React necesita volver a sincronizar tu Efecto para volver a conectar a una sala diferente.

Para **detener la sincronización,** React llamará a la función de limpieza que tu Efecto devolvió después de conectarse a la sala `"general"`. Dado que `roomId` era `"general"`, la función de limpieza se desconecta de la sala `"general"`:

```jsx
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Se conecta a la sala "general"
    connection.connect();
    return () => {
      connection.disconnect(); // Se desconecta de la sala "general" 👈
    };
    // ...
```

Luego, React ejecutará el Efecto que hayas proporcionado durante este renderizado. Esta vez, `roomId` es `"viaje"` por lo que **comenzará a sincronizar** la sala de chat `"viaje"` (hasta que su función de limpieza eventualmente también se llame):

```jsx
function ChatRoom({ roomId /* "viaje" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Se conecta a la sala "viaje" 👈
    connection.connect(); // 👈
    // ...
```

Gracias a esto, ahora estás conectado a la misma sala que el usuario eligió en la UI. ¡Desastre evitado!

Cada vez que tu componente se vuelve a renderizar con un `roomId` diferente, tu Efecto se volverá a sincronizar. Por ejemplo, digamos que el usuario cambió el `roomId` de `"viaje"` a `"música"`. React volverá a **detener la sincronización** de tu Efecto llamando a la función de limpieza (desconectándose de la sala `"viaje"`). Luego, **comenzará a sincronizarse** nuevamente al ejecutar su cuerpo con la nueva prop `roomId` (conectándose a la sala `"música"`).

Finalmente, cuando el usuario vaya a una pantalla diferente, `ChatRoom` se desmonta. Ahora no hay necesidad de permanecer conectado en absoluto. React **detendrá la sincronización** de tu Efecto por última vez y te desconectará de la sala de chat `"música"`.

### Pensar desde la perspectiva del Efecto

Recapitulemos todo lo que sucedió desde la perspectiva del componente `ChatRoom`:

1. `ChatRoom` se montó con `roomId` establecido en `"general"`
2. `ChatRoom` se actualizó con `roomId` establecido en `"viaje"`
3. `ChatRoom` se actualizó con `roomId` establecido en `"música"`
4. `ChatRoom` se desmontó

Durante cada uno de estos puntos en el ciclo de vida del componente, tu Efecto hizo diferentes cosas:

1. Tu Efecto se conectó a la sala `"general"`
2. Tu Efecto se desconectó de la sala `"general"` y se conectó a la sala `"viaje"`
3. Tu Efecto se desconectó de la sala `"viaje"` y se conectó a la sala `"música"`
4. Tu Efecto se desconectó de la sala `"música"`

Ahora pensemos qué sucedió desde la perspectiva del Efecto mismo:

```jsx
  useEffect(() => {
    // Tu Efecto se conectó a la sala especificada con el roomId...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...hasta que se desconectó
      connection.disconnect();
    };
  }, [roomId]);
```

Esta estructura de código podría inspirarte a ver lo que sucedió como una secuencia de períodos de tiempo no superpuestos:

1. Tu Efecto se conectó a la sala `"general"` (hasta que se desconectó)
2. Tu Efecto se conectó a la sala `"viaje"` (hasta que se desconectó)
3. Tu Efecto se conectó a la sala `"música"` (hasta que se desconectó)

Previamente, pensabas desde la perspectiva del componente. Cuando miraste desde la perspectiva del componente, era tentador pensar en los Efectos como «_callbacks_» o «eventos del ciclo de vida» que se disparaban en un momento específico como «después de renderizar» o «antes de desmontar». Esta forma de pensar se complica muy rápido, por lo que es mejor evitarla.

**En su lugar, siempre concéntrate en un solo ciclo de inicio/parada a la vez. No debería importar si un componente se está montando, actualizando o desmontando. Lo único que necesitas hacer es describir cómo iniciar la sincronización y cómo detenerla. Si lo haces bien, tu Efecto será resistente a ser iniciado y detenido tantas veces como sea necesario.**

Esto podría recordarte cómo no pensar si un componente se está montando o actualizando cuando escribes la lógica de renderizado que crea JSX. Describes lo que debería estar en la pantalla y React [se encarga del resto.](https://es.react.dev/learn/reacting-to-input-with-state)

### Cómo React verifica que tu Efecto pueda volver a sincronizarse [](https://es.react.dev/learn/lifecycle-of-reactive-effects#how-react-verifies-that-your-effect-can-re-synchronize "Link for Cómo React verifica que tu Efecto pueda volver a sincronizarse")

Aquí hay un ejemplo en vivo con el que puedes experimentar. Presiona «Abrir chat» para montar el componente `ChatRoom`:

**App.js**
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  return <h1>¡Bienvenido a la sala {roomId}!</h1>;
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
export function createConnection(serverUrl, roomId) {
  // Una implementación real se conectaría realmente al servidor.
  return {
    connect() {
      console.log('✅ Conectando a la sala "' + roomId + '" en ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectando de "' + roomId + '" en ' + serverUrl);
    }
  };
}
```

Observa que cuando el componente se monta por primera vez, ves tres registros:

1. `✅ Conectando a la sala "general" en https://localhost:1234...` _(solo en desarrollo)_
2. `❌ Desconectando de "general" en https://localhost:1234.` _(solo en desarrollo)_
3. `✅ Conectando a la sala "general" en https://localhost:1234...`

Los primeros dos registros son solo para desarrollo. En desarrollo, React siempre vuelve a montar cada componente una vez.

**React verifica que tu Efecto puede volver a sincronizarse forzándolo a hacerlo inmediatamente en desarrollo** Esto puede recordarte a cuando abres una puerta y la cierras una vez más para verificar si la cerradura funciona. React inicia y detiene tu Efecto una vez adicional en desarrollo para comprobar que [has implementado su limpieza adecuadamente.](https://es.react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

La razón principal por la que tu Efecto volverá a sincronizarse en la práctica es si algunos de los datos que utiliza han cambiado. En el sandbox de arriba, cambia la sala de chat seleccionada. Observa cómo, cuando cambia el valor de `roomId`, tu Efecto vuelve a sincronizarse.

Sin embargo, también hay casos más inusuales en los que es necesario que vuelva a sincronizar. Por ejemplo, intenta editar el `serverUrl` en el sandbox de arriba mientras el chat está abierto. Observa cómo el Efecto vuelve a sincronizar en respuesta a tus ediciones en el código. En el futuro, React puede agregar más características que dependan de volver a sincronizar.

### Cómo React conoce que es necesario volver a sincronizar el Efecto [](https://es.react.dev/learn/lifecycle-of-reactive-effects#how-react-knows-that-it-needs-to-re-synchronize-the-effect "Link for Cómo React conoce que es necesario volver a sincronizar el Efecto")

Podrías estarte preguntando cómo React conoce que tu Efecto necesita volverse a sincronizar luego de que el `roomId` cambia. Es porque _le dijiste a React_ que su código depende de `roomId` al incluirlo en la [lista de dependencias:](https://es.react.dev/learn/synchronizing-with-effects#step-2-specify-the-effect-dependencies)

```jsx
function ChatRoom({ roomId }) { // La prop roomId puede cambiar con el tiempo 👈
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Este Efecto lee 👈roomId 
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // Entonces le dices a React que este Efecto "depende de" roomId 👈
  // ...
```

Así es como funciona esto:

1. Sabías que `roomId` es una prop, lo que significa que puede cambiar con el tiempo.
2. Sabías que tu Efecto lee `roomId` (porque lo usas para crear la conexión).
3. Es por esto que lo especificaste como la dependencia de tu Efecto. (para que se vuelva a sincronizar cuando `roomId` cambie).

Cada vez que tu componente se vuelve a renderizar, React mirará el _array_ de dependencias que has pasado. Si alguno de los valores en el _array_ de dependencias es diferente del valor en el mismo lugar que pasaste durante el renderizado anterior, React volverá a sincronizar tu Efecto.

Por ejemplo, si pasaste `["general"]` durante el renderizado inicial, y luego pasaste `["viaje"]` durante el siguiente renderizado, React comparará `"general"` y `"viaje"`. Estos son valores diferentes (comparados con [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), por lo que React volverá a sincronizar tu Efecto. Por otro lado, si tu componente se vuelve a renderizar pero `roomId` no ha cambiado, tu Efecto permanecerá conectado a la misma sala.

### Cada Efecto representa un proceso de sincronización separado [](https://es.react.dev/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process "Link for Cada Efecto representa un proceso de sincronización separado")

Resiste la tentación de agregar lógica no relacionada a tu Efecto solo porque esta lógica necesita ejecutarse al mismo tiempo que un Efecto que ya escribiste. Por ejemplo, digamos que quieres enviar un evento de análisis cuando el usuario visita la sala. Ya tienes un Efecto que depende de `roomId`, por lo que podrías sentirte tentado a agregar la llamada de análisis allí:

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId); // 👈
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

Pero imagina que más tarde agregas otra dependencia a este Efecto que necesita restablecer la conexión. Si este Efecto se vuelve a sincronizar, también llamará a `logVisit(roomId)` para la misma sala, lo cual no pretendías. Registrar la visita **es un proceso separado** de la conexión. Escríbelos como dos Efectos separados:

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => { // 👈
    logVisit(roomId); // 👈
  }, [roomId]); // 👈

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
```

**Cada Efecto en tu código debe representar un proceso de sincronización separado e independiente.**

En el ejemplo de arriba, eliminar un Efecto no rompería la lógica del otro Efecto. Esta es una buena indicación de que sincronizan cosas diferentes, por lo que tiene sentido dividirlos. Por otro lado, si divides una pieza cohesiva de lógica en Efectos separados, el código puede verse «más limpio», pero será [más difícil de mantener.](https://es.react.dev/learn/you-might-not-need-an-effect#chains-of-computations) Es por esto que debes pensar si los procesos son iguales o diferentes, no si el código se ve más limpio.

## Los Efectos «reaccionan» a valores reactivos 

Tu Efecto lee dos variables (`serverUrl` y `roomId`), pero solo especificaste `roomId` como una dependencia:

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 👈
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // 👈
  // ...
}
```

¿Por qué no se especifica `serverUrl` como una dependencia?

Esto es porque el `serverUrl` nunca cambia debido a un rerenderizado. Siempre es el mismo sin importar cuántas veces se vuelva a renderizar el componente y por qué. Dado que `serverUrl` nunca cambia, no tendría sentido especificarlo como una dependencia. Después de todo, ¡las dependencias solo hacen algo cuando cambian con el tiempo!

Por otro lado, `roomId` puede ser diferente en un rerenderizado. **Las props, el estado y otros valores declarados dentro del componente son _reactivos_ porque se calculan durante el renderizado y participan en el flujo de datos de React.**

Si `serverUrl` fuera una variable de estado, sería reactiva. Los valores reactivos deben incluirse en las dependencias:

```jsx
function ChatRoom({ roomId }) { // Las props cambian con el tiempo
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // El estado puede cambiar con el tiempo 👈

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Tu Efecto lee props y estado 👈
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // Entonces le dices a React que este Efecto "depende de" las props y el estado 👈
  // ...
}
```

Al incluir `serverUrl` como una dependencia, te aseguras de que el Efecto se vuelva a sincronizar después de que cambie.

Intenta cambiar la sala de chat seleccionada o editar la URL del servidor en este _sandbox_:

**App.js**
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        URL del servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>¡Bienvenido a la sala {roomId}!</h1>
    </>
  );
}

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
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

**chat.js**\
```jsx
export function createConnection(serverUrl, roomId) {
  // Una implementación real en realidad se conectaría al servidor
  return {
    connect() {
      console.log('✅ Conectando a la sala "' + roomId + '" en ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado de la sala "' + roomId + '" en ' + serverUrl);
    }
  };
}
```

Cada vez que cambies un valor reactivo como `roomId` o `serverUrl`, el Efecto se vuelve a conectar al servidor del chat.

### ¿Qué significa un Efecto con dependencias vacías? 

¿Qué pasa si mueves tanto `serverUrl` como `roomId` fuera del componente?

```
const serverUrl = 'https://localhost:1234'; // 👈
const roomId = 'general'; // 👈

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ Todas las dependencias declaradas
  // ...
}
```

Ahora el código de tu Efecto no usa _ningún_ valor reactivo, por lo que sus dependencias pueden estar vacías (`[]`).

Pensando desde la perspectiva del componente, el _array_ de dependencias vacías `[]` significa que este Efecto se conecta a la sala de chat solo cuando el componente se monta, y se desconecta solo cuando el componente se desmonta. (Ten en cuenta que React aún [se volvería a sincronizar una vez más](https://es.react.dev/learn/lifecycle-of-reactive-effects#how-react-verifies-that-your-effect-can-re-synchronize) en desarrollo para probar tu lógica.)

**App.js**
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>¡Bienvenido a la sala {roomId}!</h1>;
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Cerrar chat' : 'Abrir chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom />}
    </>
  );
}
```

**chat.js**
```jsx
export function createConnection(serverUrl, roomId) {
  // Una implementación real en realidad conectaría al servidor
  return {
    connect() {
      console.log('✅ Conectando a la sala "' + roomId + '" en ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado de la sala "' + roomId + '" en ' + serverUrl);
    }
  };
}
```

Sin embargo, si [piensas desde la perspectiva del Efecto,](https://es.react.dev/learn/lifecycle-of-reactive-effects#thinking-from-the-effects-perspective) no necesitas pensar en montar y desmontar en absoluto. Lo importante es que has especificado lo que tu Efecto hace para comenzar y detener la sincronización. Hoy, no tiene dependencias reactivas. Pero si alguna vez quieres que el usuario cambie `roomId` o `serverUrl` con el tiempo (y se volverían reactivos), el código de tu Efecto no cambiará. Solo necesitarás agregarlos a las dependencias.

### Todas las variables declaradas en el cuerpo del componente son reactivas

Las props y el estado no son los únicos valores reactivos. Los valores que calculas a partir de ellos también son reactivos. Si las props o el estado cambian, tu componente se volverá a renderizar, y los valores calculados a partir de ellos también cambiarán. Es por eso que todas las variables del cuerpo del componente utilizadas por el Efecto deben estar en la lista de dependencias del Efecto.

Digamos que el usuario puede elegir un servidor de chat en el menú desplegable, pero también puede configurar un servidor predeterminado en la configuración. Supongamos que ya has puesto el estado de configuración en un [contexto](https://es.react.dev/learn/scaling-up-with-reducer-and-context) para que leas la `configuración` de ese contexto. Ahora calculas `serverUrl` en función del servidor seleccionado de las props y el servidor predeterminado:

```jsx
function ChatRoom({ roomId, selectedServerUrl }) { // roomId es reactivo
  const settings = useContext(SettingsContext); // settings es reactivo
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // serverUrl es reactivo 👈
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Tu Efecto lee roomId y serverUrl 👈
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // ¡Así que necesita volver a sincronizar cuando cualquiera de ellas cambia! 👈
  // ...
}
```

En este ejemplo, `serverUrl` no es una prop ni una variable de estado. Es una variable regular que calculas durante el renderizado. Al ser calculada durante el renderizado, puede cambiar debido a un nuevo renderizado. Es por eso que es reactiva.

**Todos los valores dentro del componente (incluidas las props, el estado y las variables en el cuerpo de tu componente) son reactivos. Cualquier valor reactivo puede cambiar en un nuevo renderizado, por lo que debes incluir los valores reactivos como dependencias del Efecto.**

En otras palabras, los Efectos «reaccionan» a todos los valores del cuerpo del componente.

#### ¿Pueden los valores globales o mutables ser dependencias?

Los valores mutables (incluidas las variables globales) no son reactivos.

Un valor mutable como location.pathname no puede ser una dependencia. Es mutable, por lo que puede cambiar en cualquier momento fuera del flujo de datos de renderizado de React. Cambiarlo no activaría un nuevo renderizado de tu componente. Por lo tanto, incluso si lo especificaras en las dependencias, React no sabría volver a sincronizar el Efecto cuando cambia. Esto también rompe las reglas de React porque leer datos mutables durante el renderizado (que es cuando calculas las dependencias) rompe la pureza del renderizado. En su lugar, debes leer y suscribirte a un valor mutable externo con useSyncExternalStore.

Un valor mutable como ref.current o cosas que lees de él tampoco pueden ser una dependencia. El objeto ref devuelto por useRef en sí puede ser una dependencia, pero su propiedad current es intencionalmente mutable. Te permite mantener un seguimiento de algo sin activar un nuevo renderizado. Pero como cambiarlo no activa un nuevo renderizado, no es un valor reactivo, y React no sabrá volver a ejecutar tu Efecto cuando cambie.

Como aprenderás a continuación en esta página, un linter verificará automáticamente estos problemas.

### React verifica que especificaste cada valor reactivo como una dependencia

Si tu linter está [configurado para React,](https://es.react.dev/learn/editor-setup#linting) verificará que cada valor reactivo utilizado por el código de tu Efecto se declare como su dependencia. Por ejemplo, este es un error de lint porque tanto `roomId` como `serverUrl` son reactivos:

**App.jsx**
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) { // roomId es reactivo
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl es reactivo

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- Algo está mal aquí!

  return (
    <>
      <label>
        URL del servidor:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>¡Bienvenido a la sala {roomId}!</h1>
    </>
  );
}

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
      <ChatRoom roomId={roomId} />
    </>
  );
}
```
**Chat.js**
```jsx
export function createConnection(serverUrl, roomId) {
  // Una implementación real en realidad se conectaría al servidor
  return {
    connect() {
      console.log('✅ Conectando a la sala "' + roomId + '" en ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Desconectado de la sala "' + roomId + '" en ' + serverUrl);
    }
  };
}
```

Esto puede parecer un error de React, pero en realidad React está señalando un error en tu código. Tanto `roomId` como `serverUrl` pueden cambiar con el tiempo, pero olvidaste volver a sincronizar tu Efecto cuando cambian. Seguirás conectado a la `roomId` y `serverUrl` iniciales incluso después de que el usuario elija valores diferentes en la interfaz de usuario.

Para solucionar el error, sigue la sugerencia del linter de especificar `roomId` y `serverUrl` como dependencias de tu Efecto:

```jsx
function ChatRoom({ roomId }) { // roomId es reactivo
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl es reactivo
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // ✅ Todas las dependencias están declaradas 👈
  // ...
}
```

Intenta esta solución en el _sandbox_ de arriba. Verifica que el error del linter haya desaparecido y que el chat se vuelva a conectar cuando sea necesario.

> [!note]
> En algunos casos, React _sabe_ que un valor nunca cambia aunque se declare dentro del componente. Por ejemplo, la función [`set`](https://es.react.dev/reference/react/useState#setstate) devuelta por `useState` y el objeto ref devuelto por [`useRef`](https://es.react.dev/reference/react/useRef) son _estables_—se garantiza que no cambiarán en un nuevo renderizado. Los valores estables no son reactivos, por lo que puedes omitirlos de la lista. Incluirlos está permitido: no cambiarán, por lo que no importa.

### ¿Qué hacer cuando no quieres volver a sincronizar? 

En los ejemplos previos, has arreglado el error del linter enumerando `roomId` y `serverUrl` como dependencias.

**Sin embargo, podrías en cambio «demostrar» al linter que estos valores no son reactivos,** es decir, que _no pueden_ cambiar como resultado de un nuevo renderizado. Por ejemplo, si `serverUrl` y `roomId` no dependen del renderizado y siempre tienen los mismos valores, puedes moverlos fuera del componente. Ahora no necesitan ser dependencias:

```jsx
const serverUrl = 'https://localhost:1234'; // serverUrl no es reactivo 👈
const roomId = 'general'; // roomId no es reactivo 👈

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ Declaradas todas las dependencias 👈
  // ...
}
```

También puedes moverlos _dentro del Efecto._ No se calculan durante el renderizado, por lo que no son reactivos:

```jsx
function ChatRoom() {
  useEffect(() => {
    const serverUrl = 'https://localhost:1234'; // serverUrl no es reactivo
    const roomId = 'general'; // roomId no es reactivo
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ Declaradas todas las dependencias
  // ...
}
```

**Los Efectos son bloques de código reactivos.** Se vuelven a sincronizar cuando los valores que lees dentro de ellos cambian. A diferencia de los controladores de eventos, que solo se ejecutan una vez por interacción, los Efectos se ejecutan cada vez que es necesaria la sincronización.

**No puedes «elegir» tus dependencias.** Tus dependencias deben incluir cada [valor reactivo](https://es.react.dev/learn/lifecycle-of-reactive-effects#all-variables-declared-in-the-component-body-are-reactive) que lees en el Efecto. El linter hace cumplir esto. A veces esto puede conducir a problemas como bucles infinitos y a que tu Efecto se vuelva a sincronizar demasiado a menudo. ¡No soluciones estos problemas suprimiendo el linter! Esto es lo que debes intentar en su lugar:

- **Verifica que tu Efecto represente un proceso de sincronización independiente.** Si tu Efecto no sincroniza nada, [podría ser innecesario.](https://es.react.dev/learn/you-might-not-need-an-effect) Si sincroniza varias cosas independientes, [divídelo.](https://es.react.dev/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process)
    
- **Si quieres leer la última versión de las props o el estado sin «reaccionar» a ellas y volver a sincronizar el Efecto,** puedes dividir tu Efecto en una parte reactiva (que mantendrás en el Efecto) y una parte no reactiva (que extraerás en algo llamado un _Evento de Efecto_). [Lee sobre cómo separar los Eventos de los Efectos.](https://es.react.dev/learn/separating-events-from-effects)
  
- **Evita confiar en objetos y funciones como dependencias.** Si creas objetos y funciones durante el renderizado y luego los lees desde un Efecto, serán diferentes en cada renderizado. Esto hará que tu Efecto se vuelva a sincronizar cada vez. [Lee más sobre cómo eliminar las dependencias innecesarias de los Efectos.](https://es.react.dev/learn/removing-effect-dependencies)
  

> [!warning]
>El linter es tu amigo, pero sus poderes son limitados. El linter solo sabe cuando las dependencias son _incorrectas_. No sabe la _mejor_ manera de resolver cada caso. Si el linter sugiere una dependencia, pero agregarla causa un bucle, no significa que el linter deba ser ignorado. Necesitas cambiar el código dentro (o fuera) del Efecto para que ese valor no sea reactivo y no _necesite_ ser una dependencia.
>
Si tienes una base de código existente, es posible que tengas algunos Efectos que supriman el >linter de esta manera:
>
>```jsx
>useEffect(() => {
>  // ...
>  // 🔴 Evita suprimir el linter de esta manera:
>  // eslint-ignore-next-line react-hooks/exhaustive-deps
>}, []);
>```
>
>En la [siguiente](https://es.react.dev/learn/separating-events-from-effects) [página](https://es.react.dev/learn/removing-effect-dependencies), aprenderás cómo arreglar este código sin romper las reglas. ¡Siempre vale la pena arreglarlo!

## Recapitulación

- Los componentes pueden montarse, actualizarse y desmontarse.
- Cada Efecto tiene un ciclo de vida separado del componente circundante.
- Cada Efecto describe un proceso de sincronización separado que puede _iniciarse_ y _detenerse_.
- Cuando escribes y lees Efectos, piensa desde la perspectiva de cada Efecto individual (cómo iniciar y detener la sincronización) en lugar de desde la perspectiva del componente (cómo se monta, actualiza o desmonta).
- Valores declarados dentro del cuerpo del componente son «reactivos».
- Valores reactivos deben volver a sincronizar el Efecto porque pueden cambiar con el tiempo.
- El linter verifica que todos los valores reactivos usados dentro del Efecto estén especificados como dependencias.
- Todas las banderas de error del linter son legítimas. Siempre hay una manera de arreglar el código para que no rompa las reglas.