Algunos componentes tienen la necesidad de sincronizarse con sistemas externos. Por ejemplo, es posible que desees controlar un componente que no sea de React en función a un estado de React, configurar una conexión de servidor, o enviar un registro de análisis cuando un componente se muestra en la pantalla. **Los _Efectos_ te permiten ejecutar código después del renderizado para que puedas sincronizar tu componente con un sistema fuera de React**.

### Aprenderás

- Qué son los Efectos
- Cómo se diferencian los Efectos de los eventos
- Cómo declarar un Efecto en tu componente
- Cómo evitar que un Efecto se vuelva a ejecutar innecesariamente
- Por qué los Efectos se ejecutan dos veces en desarrollo y cómo arreglarlo

## ¿Qué son los efectos y en que se diferencian de los eventos?

Antes de empezar con los Efectos, necesitas familiarizarte con dos tipos de lógica dentro de los componentes de React:

- **Código renderizado** (introducido en [Describir la UI](https://es.react.dev/learn/describing-the-ui)) se encuentra en el nivel superior de tu componente. Aquí es donde tomas las props y el estado, los modificas, y se devuelve el JSX que se desea ver en la pantalla. [El código renderizado debe ser puro.](https://es.react.dev/learn/keeping-components-pure) Como si fuese una fórmula matemática, sólo debe _calcular_ el resultado, y no hacer nada más.
  
- **Controladores de eventos** (introducido en [Añadir interactividad](https://es.react.dev/learn/adding-interactivity)) son funciones anidadas dentro de tus componentes que _hacen_ cosas en lugar de solo calcularlas. Un controlador de evento podría actualizar un campo de un formulario, enviar una solicitud HTTP POST para comprar un producto, o hacer que el usuario navegue hacia otra pantalla. **Los controladores de eventos contienen [«efectos secundarios»](https://es.wikipedia.org/wiki/Efecto_secundario_(inform%C3%A1tica)) (Pueden cambiar el estado del programa) causado por una acción específica del usuario (por ejemplo, al hacer clic en un botón o al escribir)**.
  

**A veces, esto no es suficiente**. Considera un componente `ChatRoom` que debe conectarse al servidor del chat cada vez que esté visible en pantalla. Conectarse al servidor no es un cálculo puro (es un efecto secundario), por lo que no puede suceder durante el renderizado. Sin embargo, no hay un evento particular como un clic que haga que `ChatRoom` se muestre en pantalla.

**Los _Efectos_ te permiten especificar efectos secundarios que son causados por el renderizado en sí mismo, en lugar de por un evento particular.** Enviar un mensaje en el chat es un _evento_ porque es directamente causado por el usuario haciendo clic en un botón. Sin embargo, **establecer una conexión a un servidor es un _Efecto_ porque debería suceder sin importar qué interacción causó que el componente apareciera. Los efectos se ejecutan al final de la [confirmación](https://es.react.dev/learn/render-and-commit), después de que la pantalla se actualice**. Este es un buen momento para sincronizar los componentes de React con algún sistema externo (como una red o una biblioteca de terceros).

> [!note]
> A partir de ahora en este texto, «_Efecto_» en mayúsculas se refiere a la definición específica de React mencionada anteriormente, es decir, un efecto secundario causado por el renderizado. Para referirnos al concepto de programación más amplio, diremos «_efecto secundario_».

## Quizás no necesites un Efecto 

**No te apresures en añadir Efectos en tus componentes.** Ten en cuenta que los Efectos se usan típicamente para «_salir_» de tu código React y sincronizar con algún sistema _externo_. Esto incluye APIs del navegador, widgets de terceros, red, etc. Si tu Efecto solo ajusta un estado basado en otro estado, [quizás no necesites un Efecto.](https://es.react.dev/learn/you-might-not-need-an-effect)

## Cómo escribir un Efecto

Para escribir un Efecto, sigue los siguientes pasos:

1. **Declara un Efecto.** Por defecto, tu Efecto se ejecutará después de cada [_confirmación_](https://es.react.dev/learn/render-and-commit).
2. **Define las dependencias del Efecto.** La mayoría de los Efectos solo deben volver a ejecutarse _cuando sea necesario_ en lugar de hacerlo después de cada renderizado. Por ejemplo, una animación de desvanecimiento solo debe desencadenarse cuando aparece el componente. La conexión y desconexión a una sala de chat solo debe suceder cuando el componente aparece y desaparece, o cuando cambia la sala de chat. Aprenderás cómo controlar esto especificando las _dependencias_.
3. **Añade limpieza si es necesario.** Algunos Efectos necesitan especificar cómo detener, deshacer, o limpiar cualquier cosa que estaban haciendo. Por ejemplo, «conectar» necesita «desconectar», «suscribirse» necesita «anular suscripción» y «buscar» necesita «cancelar» o «ignorar». Aprenderás cómo hacer esto devolviendo una _función de limpieza_

Veamos cada uno de estos pasos en detalle.

### ⭐ Paso 1: Declara un Efecto

Para declarar un efecto en tu componente, importa el [Hook `useEffect`](https://es.react.dev/reference/react/useEffect) desde React:

```jsx
import { useEffect } from 'react';
```

Luego, llámalo en el nivel superior de tu componente y escribe algún código dentro del Efecto:

```jsx
function MyComponent() {
  useEffect(() => { // 👈
    // El código aquí se ejecutará después de *cada* renderizado
  });
  return <div />;
}
```

**Cada vez que el componente se renderiza, React actualizará la pantalla _y entonces_ ejecutará el código dentro de `useEffect`**. En otras palabras, **`useEffect` «_retrasa_» la ejecución de una parte del código hasta que el renderizado es reflejado en la pantalla.**

Veamos cómo puedes usar un _Efecto_ para sincronizarlo con un sistema externo. Considera un componente de React `<VideoPlayer>`. Sería bueno controlar si está reproduciéndose o en pausa, enviándole la prop `isPlaying`:

```jsx
<VideoPlayer isPlaying={isPlaying} />;
```

Tu componente personalizado `VideoPlayer` renderiza la etiqueta incorporada en el navegador, [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video):

```jsx
function VideoPlayer({ src, isPlaying }) {
  // TODO: hacer algo con isPlaying
  return <video src={src} />;
}
```

Sin embargo, la etiqueta de navegador `<video>` no tiene la prop `isPlaying`. La única manera de controlarlo es la de llamar manualmente a los métodos [`play()`](https://developer.mozilla.org/es/docs/Web/API/HTMLMediaElement/play) y [`pause()`](https://developer.mozilla.org/es/docs/Web/API/HTMLMediaElement/pause) en el elemento DOM. **Necesitas sincronizar el valor de la prop `isPlaying`, que indica si el video _debería_ estar reproduciéndose en ese momento, con llamadas como `play()` y `pause()`.**

Primero necesitaremos [obtener una referencia](https://es.react.dev/learn/manipulating-the-dom-with-refs) al nodo de DOM `<video>`.

==Puede que sientas la tentación de intentar llamar a `play()` o `pause()` durante el renderizado, pero eso no es correcto==:

**App.js**
```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // Llamar a estas funciones mientras se renderiza no está permitido. ❌
  } else {
    ref.current.pause(); // Esto también causa error. ❌
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

Muestra:

![[3-sincronizar-con-efectos-1.png]]

La razón por la que **este código no es correcto es porque intenta hacer algo con el nodo de DOM durante el renderizado**. En React, [renderizar debe ser un cálculo puro](https://es.react.dev/learn/keeping-components-pure) de JSX y no debe contener efectos secundarios como la modificación del DOM.

Además, **cuando se llama a `VideoPlayer` por primera vez, ¡su DOM no existe todavía! No hay un nodo de DOM para llamar a `play()` o `pause()`**, porque React no sabe qué DOM crear hasta que se devuelva el JSX.

**La solución es envolver el efecto secundario con `useEffect` para sacarlo del cálculo de renderizado:**

```jsx
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => { 👈
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

**Al envolver la actualización del DOM en un Efecto, permites que React actualice la pantalla primero. Luego se ejecuta tu Efecto**.

Cuando tu componente `VideoPlayer` se renderiza (ya sea por primera vez o si se vuelve a renderizar), algunas cosas van a suceder. Primero, React actualizará la pantalla, asegurándose que la etiqueta `<video>` está en el DOM con las props correctas. Luego, React ejecutará tu Efecto. Por último, tu Efecto llamará a `play()` o `pause()` dependiendo del valor `isPlaying`.

Presiona _Reproducir_/_Pausar_ múltiples veces y observa como el reproductor de video se mantiene sincronizado con el valor de `isPlaying`:

**App.js**
```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {// 👈
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pausar' : 'Reproducir'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

Muestra:

![[2.1-manipular-el-dom-con-refs-1.png]]

Al pulsar sobre el botón se reproduce el video y cambia el texto:

![[2.1-manipular-el-dom-con-refs-2.png]]

En este ejemplo, el «sistema externo» que sincronizaste con el estado de React fue la API _browser media_. Puedes usar un enfoque similar para envolver código heredado que no es de React (como plugins de jQuery) en componentes declarativos de React.

Nota que en la práctica, controlar un reproductor de video es mucho más complejo. Llamar a `play()` puede fallar, el usuario podría reproducir o pausar usando los controles integrados del navegador, etc. Este ejemplo es muy simplificado e incompleto.

> [!warning]
>Por defecto, los Efectos se ejecutan después de _cada_ renderizado. Por lo tanto un código así **producirá un ciclo infinito:**
>
>```jsx
>const [count, setCount] = useState(0);
>useEffect(() => {
>  setCount(count + 1);
>});
>```
>
>**Los Efectos se ejecutan como _resultado_ del renderizado. Establecer el estado _desencadena_ el renderizado**. Establecer el estado inmediatamente en un Efecto es como enchufar un enchufe a sí mismo. El Efecto se ejecuta, establece el estado, lo que provoca un nuevo renderizado, lo que hace que el efecto se ejecute, establece el estado de nuevo, esto provoca otro renderizado, y así sucesivamente.
>
>**Los Efectos por lo general deben sincronizar tus componentes con un sistema _externo_. Si no hay ningún sistema externo y solo deseas ajustar algún estado en base a otro estado [quizás no necesites un Efecto.](https://es.react.dev/learn/you-might-not-need-an-effect)**

### ⭐ Paso 2: Define las dependencias del Efecto 

**Por defecto, los Efectos se ejecutan después de _cada_ renderizado**. **A menudo, esto no es lo que tu buscas**:

- **A veces, es lento.** Sincronizar con un sistema externo no siempre es instantáneo, por lo que es posible que desees evitar hacerlo a menos que sea necesario. **Por ejemplo, no quieres volver a conectarte al servidor de chat en cada pulsación de tecla**.
- **A veces, está mal**. Por ejemplo, **no quieres desencadenar una animación de desvanecimiento en un componente en cada pulsación de tecla**. La animación solo se debe reproducir cuando el componente aparece por primera vez.

Para demostrar el problema, aquí está el ejemplo anterior con algunas llamadas `console.log` y un campo de texto que actualiza el estado del componente padre. Observa cómo escribir provoca que el Efecto se ejecute de nuevo:

**App.js**
```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Llamando a video.play()');
      ref.current.play();
    } else {
      console.log('Llamando a video.pause()');
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pausa' : 'Reproducir'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

Muestra sólo dos logs en la consola:

![[3-sincronizar-con-efectos-2.png]]

Pero al escribir la palabra 'hola' en el input, tenemos 4 logs más (una por cada letra escrita):

![[3-sincronizar-con-efectos-3.png]]

**Puedes indicarle a React omitir la innecesaria ejecución del Efecto especificando un array de _dependencias_ como segundo argumento en la llamada a `useEffect`. Empieza añadiendo un array vacío `[]`** en el ejemplo anterior en la línea 14:

```jsx
useEffect(() => {
	// ...
}, []);
```

Al hacer esto, deberías ver el error `React Hook useEffect has a missing dependency: 'isPlaying'` (Al Hook de React useEffect le falta una dependencia: ‘isPlaying’):

**App.js**
```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Llamando a video.play()');
      ref.current.play();
    } else {
      console.log('Llamando a video.pause()');
      ref.current.pause();
    }
  }, []); // Esto causa un error 👈❌

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pausa' : 'Reproducir'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```  

Muestra:

![[3-sincronizar-con-efectos-4.png]]

**El problema es que el código dentro de tu Efecto _depende de_ la prop `isPlaying` para decidir qué hacer, pero esta dependencia no se declaró explícitamente. Para solucionar este problema, añade `isPlaying` al array de dependencias**:

```jsx
  useEffect(() => {
    if (isPlaying) { // Se usa aquí... 👈
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ...entonces debe ser declarado aquí!👈
```

**Ahora que todas las dependencias fueron declaradas, no habrá errores**. Definiendo `[isPlaying]` como dependencia del array, le indica a React que debe saltarse la ejecución del Efecto si `isPlaying` tiene el mismo valor que el renderizado anterior. Con este cambio, escribir en el campo de texto no causa que el Efecto se vuelva a ejecutar, pero presionar Reproducir/Pausar sí lo hace:

**App.js**
```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Llamando a video.play()');
      ref.current.play();
    } else {
      console.log('Llamando a video.pause()');
      ref.current.pause();
    }
  }, [isPlaying]); // 👈

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pausa' : 'Reproducir'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

Muestra:

![[3-sincronizar-con-efectos-2.png]]

Al escribir en el input, ya no se ejecuta el Efecto, por lo que no se muestran más mensajes por consola:

![[3-sincronizar-con-efectos-5.png]]

El array de dependencias puede contener múltiples dependencias. ==**React sólo saltará la ejecución del Efecto si todas las dependencias que especifiques tienen exactamente los mismos valores que tenían durante el renderizado anterior**==. React compara los valores de dependencia utilizando la comparación [`Object.is`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Consulta la [referencia de `useEffect`](https://es.react.dev/reference/react/useEffect#reference) para más detalles.

**Observa que tú no puedes «elegir» las dependencias. Obtendrás un error de lint si las dependencias que definiste no coinciden con lo que React espera según el código dentro de tu Efecto**. Esto ayuda a detectar errores en tu código. Si no deseas que cierto código se vuelva a ejecutar, [_edita el código del Efecto_ para no «necesitar» esa dependencia.](https://es.react.dev/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)

>[!important]
>El comportamiento sin un array de dependencias y con un array de dependencias vacío `[]` son diferentes:
>
>```
>useEffect(() => {
>  // Esto se ejecuta después de cada renderizado
>});
>
>useEffect(() => {
>  // Esto sólo se ejecuta en el montaje (cuando el componente aparece)
>}, []);
>
>useEffect(() => {
>  // Esto se ejecuta en el montaje *y también* si a o b han cambiado desde el último renderizado
>}, [a, b]);
>```
>
>En el próximo paso analizaremos lo que significa «montaje».

#### ¿Por qué se omitió la ref del array de dependencias?

**Este efecto utiliza tanto `ref` como `isPlaying`, pero solo `isPlaying` es declarado como una dependencia**:

```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]);
```

Esto se debe a que el objeto ref tiene una identidad estable: React garantiza que siempre obtendrás el mismo objeto a partir de la misma llamada a useRef en cada renderizado. **Nunca cambia, por lo que por sí solo nunca hará que el Effect se vuelva a ejecutar. Por lo tanto, no importa si lo incluyes o no**. Incluirlo también está bien:

```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying, ref]); // 👈
```
  
**La función `set` devuelta por `useState` también tiene identidad estable, así que a menudo verás que también es omitida de las dependencias. ==Si el linter te permite omitir una dependencia sin errores, entonces es seguro hacerlo==.**

**Omitir dependencias que siempre son estables solo funciona cuando el linter puede «ver» que el objeto es estable. Por ejemplo, ==si ref se pasó desde un componente padre, tienes que declararlo en el array de dependencias==**. Sin embargo, esto es bueno porque no se puede saber si el componente padre siempre pasa la mismo ref, o pasa una de varias refs condicionalmente. Por lo tanto, tu Efecto dependería de qué ref se pasa.

### ⭐ Paso 3: Añade limpieza si es necesario 

Considera otro ejemplo. Estás escribiendo un componente `ChatRoom` que necesita conectarse al servidor del chat cuando aparece. Se te da una API `createConnection()` que devuelve un objeto con los métodos `connect()` y `disconnect()`. ¿Cómo mantienes conectado el componente mientras este se muestra al usuario?

Comienza por escribir la lógica del Efecto:

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

Sería lento conectarse al chat después de cada nuevo renderizado, así que añades el array de dependencias:

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []); // 👈
```

**El código dentro del Efecto no usa ninguna prop o estado, por lo que el array de dependencias está vacío `[]`. Esto le indica a React que solo ejecute este código cuando se «monta» el componente, es decir, aparece en pantalla por primera vez.**

Ejecutemos este código:

**App.js**
```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => { // 👈
    const connection = createConnection();
    connection.connect();
  }, []);  // 👈
  return <h1>¡Bienvenido al chat!</h1>;
}
```

**chat.js**
```jsx
export function createConnection() {
  // Una implementación real se conectaría al servidor
  return {
    connect() {
      console.log('✅ Conectando...');
    },
    disconnect() {
      console.log('❌ Desconectado.');
    }
  };
}
```

Muestra:

![[3-sincronizar-con-efectos-6.png]]

Este efecto solo se ejecuta cuando se monta el componente, entonces podrías pensar que `"✅ Conectando..."` se imprime una vez en la consola. **Sin embargo, si revisas la consola, `"✅ Conectando..."` se imprime dos veces. ¿Por qué sucede esto?**

Imagina que el componente `ChatRoom` es parte de una gran aplicación con muchas pantallas diferentes. El usuario inicia su viaje en la página `ChatRoom`. El componente se monta y llama a `connection.connect()`. Entonces imagina que el usuario navega hacia otra pantalla, por ejemplo, a la página de Configuración. **El componente `ChatRoom` se desmonta. Finalmente, el usuario hace clic en el botón de atrás y `ChatRoom` se monta nuevamente. Esto configuraría una segunda conexión ¡Pero la primera conexión nunca fue destruida! A medida que el usuario navega por la aplicación, las conexiones seguirían acumulándose**.

Errores como este son fáciles de pasarlos por alto sin una extensa prueba manual. **Para ayudarte a detectarlos rápidamente, en desarrollo, ==React vuelve a montar cada componente una vez inmediatamente después de su montaje inicial==**.

Ver en consola dos veces `"✅ Conectando..."` te ayuda a notar el problema real: tu código no cierra la conexión cuando el componente se desmonta.

Para solucionar este problema, devuelve una _función de limpieza_ desde el Efecto:

```jsx
    useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => { // 👈
      connection.disconnect();
    };
  }, []);
```

**React llamará a la función de limpieza antes que se ejecute el Efecto nuevamente, y una última vez cuando el componente se desmonta (se remueve)**. Veamos qué sucede cuando se implementa la función de limpieza:

**App.js**
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>¡Bienvenido al chat!</h1>;
}
```

**chat.js**
```jsx
export function createConnection() {
  // Una aplicación real se conectaría al servidor
  return {
    connect() {
      console.log('✅ Conectando...');
    },
    disconnect() {
      console.log('❌ Desconectado.');
    }
  };
}
```

Muestra:

![[3-sincronizar-con-efectos-7.png]]

Ahora obtendrás tres mensajes en la consola en el modo de desarrollo:

1. `"✅ Conectando..."`
2. `"❌ Desconectado."`
3. `"✅ Conectando..."`

**==Este es el comportamiento correcto en modo de desarrollo==. Al volver a montar el componente, React verifica que navegar a otro lado y luego volver, no romperá tu código. ¡Desconectar y luego conectar nuevamente es exactamente lo que debería suceder!** Cuando implementas la limpieza adecuadamente, no debe haber ninguna diferencia visible para el usuario entre ejecutar el Efecto una vez o ejecutarlo, limpiarlo y volver a ejecutarlo. Hay llamadas adicionales a connect/disconnect porque React está explorando tu código en busca de errores en desarrollo. **Esto es normal, ¡No intentes hacerlo desaparecer!**.

**En producción, solo verás `"✅ Conectando..."` una vez.** Volver a montar componentes solo sucede en desarrollo para ayudarte a encontrar Efectos que necesitan limpieza. Puedes desactivar el [Modo Estricto](https://es.react.dev/reference/react/StrictMode) para optar por el comportamiento de producción, pero recomendamos dejarlo activado. Esto te permite encontrar muchos errores como el anterior.

## ¿Cómo manejar que el Efecto se ejecute dos veces en desarrollo?

React intencionalmente vuelve a montar tus componentes en desarrollo con el fin de encontrar errores como en el anterior ejemplo. **La pregunta correcta no es «¿Cómo ejecutar el Efecto una sola vez?», sino «¿Cómo arreglar mi Efecto para que funcione después de que se vuelva a montar?»**

**Usualmente, la respuesta es implementar una función de limpieza. La función de limpieza debería detener o deshacer lo que sea que el Efecto hacía**. Generalmente, la respuesta es implementar la función de limpieza. La regla general es que el usuario no deba ser capaz de distinguir si el Efecto se ejecutó una sola vez (como en producción) o en una secuencia _configurar → limpiar → configurar_ (como se vería en desarrollo).

La gran parte de los Efectos que escribas se ajustan a uno de los patrones comunes que se describen a continuación.


### ⭐ No uses _refs_ para evitar que se disparen los Efectos.

Un error común a la hora de evitar que los efectos se activen dos veces durante el desarrollo es utilizar a `ref` para evitar que el efecto se ejecute más de una vez. Por ejemplo, podrías «_corregir_» el error anterior con un `useRef`:

```jsx
  const connectionRef = useRef(null); // 👈
  useEffect(() => {
    // 🚩 This wont fix the bug!!! // 👈
    if (!connectionRef.current) { // 👈
      connectionRef.current = createConnection();
      connectionRef.current.connect();
    }
  }, []);
```

Esto hace que solo se vea `"✅ Connecting..."` una vez en desarrollo, pero no soluciona el error.

Cuando el usuario se aleja, la conexión aún no se cierra y cuando regresa, se crea una nueva conexión. A medida que el usuario navega por la aplicación, las conexiones se seguirán acumulando, igual que antes del «arreglo».

Para corregir el error, no basta con hacer que el efecto se ejecute una vez. El efecto debe funcionar después de volver a montarlo, lo que significa que la conexión debe limpiarse como en la solución anterior.

Consulte los ejemplos siguientes para saber cómo manejar patrones comunes.
### Controlar widgets que no son de React 

A veces necesitas añadir widgets UI que no estén escritos en React. Por ejemplo, digamos que añades un componente de mapa a tu página y tiene un método `setZoomLevel()` y te gustaría mantener el nivel de zoom sincronizado con una variable de estado `zoomLevel` en tu código de React. Tu Efecto se vería similar a esto:

```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

**Ten en cuenta que en este caso no se necesita limpiar nada. En desarrollo, React llamará al Efecto dos veces, pero esto no es un problema porque llamar a `setZoomLevel` dos veces con el mismo valor no hace nada**. Puede ser un poco más lento, pero no importa porque no se montará innecesariamente en producción.

Algunas APIs pueden no permitir que se las llame dos veces seguidas. Por ejemplo, el método [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) del elemento integrado [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) arrojará una excepción si se le llama dos veces. Implementa la función de limpieza para que cierre el elemento `<dialog>`.

```jsx
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close(); // 👈
}, []);
```

En desarrollo, tu Efecto va a llamar a `showModal()`, e inmediatamente a `close()`, y después a `showModal()` de nuevo. Esto tiene el mismo comportamiento visible para el usuario que llamar a `showModal()` una vez, como se vería en producción.

### ⭐ Suscribirse a eventos

**Si tu Efecto se suscribe a algo, la función de limpieza debería anular la suscripción**:

```jsx
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll); // 👈
}, []);
```

En desarrollo, tu Efecto va a llamar a `addEventListener()`, e inmediatamente a `removeEventListener()`, y después a `addEventListener()` de nuevo con el mismo controlador de evento. Por lo tanto, solo habría una suscripción activa a la vez. Esto tiene el mismo comportamiento visible para el usuario que llamar a `addEventLListener()` una vez, como en producción.

### Desencadenar animaciones 

Si tu Efecto realiza alguna animación, la función de limpieza debería reiniciar la animación a los valores iniciales:

```jsx
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Desencadena la animación
  return () => { // 👈
    node.style.opacity = 0; // Reinicia a los valores iniciales // 👈
  };
}, []); // 👈
```

En desarrollo, la opacidad se establecerá en `1`, luego en `0` y luego nuevamente en `1`. Esto debería tener el mismo comportamiento visible para el usuario que establecerlo directamente en `1`, como en producción. Si usas una biblioteca de animación de terceros con soporte para interpolación, la función de limpieza debería reiniciar la línea de tiempo a su estado inicial.

### ⭐ Obtención de datos

**Si tu Efecto realiza una petición de algo, la función de limpieza debe [cancelar la petición](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) o ignorar su resultado**:

```jsx
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) { // 👈
      setTodos(json);
    }
  }

  startFetching();

  return () => { // 👈
    ignore = true; // 👈
  };
}, [userId]); // 👈
```

No puedes «cancelar» la petición que ya ocurrió, pero tu función de limpieza debería garantizar que la búsqueda que ya _no es relevante_ no siga afectando tu aplicación. Si el `userId` cambia de `'Alice'` a `'Bob'`, la limpieza garantiza que la función de `'Alice'` sea ignorada incluso si llega después de `'Bob'`.

**En desarrollo, verás dos solicitudes en la pestaña de Red.** No hay nada de malo en eso. Con el enfoque anterior, el primer Efecto se limpiará inmediatamente, por lo que su copia de la variable `ignore` se establecerá en `true`. Entonces, aunque haya una solicitud adicional, no afectará al estado gracias a la verificación `if (!ignore)`.

**En producción, solo habrá una solicitud.** Si la segunda solicitud en desarrollo te molesta, el mejor enfoque es utilizar una solución que deduplique las solicitudes y almacene en caché sus respuestas entre componentes:

```jsx
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

Esto no solo mejorará la experiencia de desarrollo, sino que también hará que tu aplicación se sienta más rápida. Por ejemplo, el usuario que presiona el botón atrás no tendrá que esperar a que se carguen los datos nuevamente porque estarán en caché. Puedes construir esta caché tú mismo o utilizar una de las muchas alternativas a la obtención de datos manual con Efectos.

#### ⭐ ¿Cuáles son las mejores alternativas a la obtención de datos con Efectos? 

Escribir llamadas `fetch` dentro de Efectos es una forma [popular de obtener datos](https://www.robinwieruch.de/react-hooks-fetch-data/), especialmente en aplicaciones totalmente del lado del cliente. Sin embargo, este es un enfoque muy manual y tiene importantes desventajas:

- **Los Efectos no se ejecutan en el servidor.** Esto significa que el HTML renderizado inicialmente en el servidor solo incluirá un estado de carga sin datos. El ordenador del cliente tendrá que descargar todo el JavaScript y renderizar tu aplicación solo para descubrir que ahora necesita cargar los datos. Esto no es muy eficiente.
- **La obtención de datos directamente en Efectos facilita la creación de «cascadas de red» (_network waterfalls_).** Se renderiza el componente padre, se obtienen algunos datos, luego se renderizan los componentes hijos, y luego ellos hacen lo mismo. Si la red no es muy rápida, este proceso secuencial es significativamente más lento que obtener todos los datos en paralelo de una sola vez.
- **La obtención de datos directamente en Efectos suele significar que no se precargan ni se almacenan en caché los datos.** Por ejemplo, si el componente se desmonta y se vuelve a montar, tendría que obtener los datos de nuevo.
- **No es muy ergonómico.** Hay bastante código boilerplate al hacer llamadas `fetch` de tal manera que no sufra de errores como las [condiciones de carrera.](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)

Esta lista de inconvenientes no es específica de React. Se aplica a la obtención de datos en el montaje con cualquier biblioteca. Al igual que con el enrutamiento, la obtención de datos no es trivial para hacerlo bien, por lo que recomendamos los siguientes enfoques:

- **Si usas un [framework](https://es.react.dev/learn/start-a-new-react-project#production-grade-react-frameworks), utiliza su mecanismo de obtención de datos integrado.** Los frameworks modernos de React han integrado mecanismos de obtención de datos que son eficientes y no sufren los inconvenientes anteriores.
- **De lo contrario, considera la posibilidad de utilizar o construir una caché del lado del cliente.** Las soluciones populares de código abierto incluyen [React Query](https://tanstack.com/query/latest), [useSWR](https://swr.vercel.app/), y [React Router 6.4+.](https://beta.reactrouter.com/en/main/start/overview) También puedes crear tu propia solución, en cuyo caso se usarían Efectos por debajo, pero también se añadiría lógica para deduplicar las peticiones, almacenar en caché las respuestas y evitar las cascadas de red (precargando los datos o elevando los requisitos de datos a las rutas).

Puedes seguir obteniendo datos directamente en Efectos si ninguno de estos enfoques te conviene.

### Enviar analítica 

Considera este código que envía un evento de análisis en la visita a la página:

```jsx
useEffect(() => {
  logVisit(url); // Enviar una solicitud POST 👈
}, [url]);
```

En desarrollo, `logVisit` será llamado dos veces para cada URL, por lo que podrías sentirte tentado a solucionar eso. **Recomendamos mantener este código tal como está.** Como en los ejemplos anteriores, no hay una diferencia de comportamiento _visible para el usuario_ entre correrlo una vez y correrlo dos veces. Desde un punto de vista práctico, `logVisit` no debería hacer nada en desarrollo porque no quieres que los registros de las máquinas de desarrollo afecten las métricas de producción. Tu componente se vuelve a montar cada vez que guardas el archivo, por lo que de todas formas registra visitas extras en desarrollo.

**En producción, no va a haber registro de visitas duplicados.**

Para depurar los eventos de análisis que estás enviando, puedes implementar tu aplicación en un entorno de pruebas (que se ejecuta en modo producción) o temporalmente desactivar el [Modo estricto](https://es.react.dev/reference/react/StrictMode) y su control de remontaje en desarrollo. También puedes enviar análisis desde los controladores de eventos de cambio de ruta en lugar de Efectos. Para análisis más precisos, la [API Observador de Intersección](https://developer.mozilla.org/es/docs/Web/API/Intersection_Observer_API) puede ayudar a rastrear qué componentes están en la vista y cuánto tiempo permanecen visibles.

### ⭐ No es un Efecto: Inicializar la aplicación

Algunas lógicas solo deben ejecutarse una vez al inicio de la aplicación. Puedes colocarlas fuera de tus componentes:

```jsx
if (typeof window !== 'undefined') { // Comprueba si estamos ejecutando en el navegador. 👈
  checkAuthToken(); // 👈
  loadDataFromLocalStorage(); // 👈
}

function App() {
  // ...
}
```

Esto garantiza que la lógica se ejecute solo una vez después de que el navegador cargue la página.

### No es un Efecto: Comprar un producto

A veces, incluso si escribes una función de limpieza, no hay forma de evitar las consecuencias visibles para el usuario de ejecutar el Efecto dos veces. Por ejemplo, tal vez tu Efecto envíe una solicitud POST, como comprar un producto:

```jsx
useEffect(() => {
  // 🔴 Está mal: Este Efecto se ejecuta dos veces en desarrollo, exponiendo un problema en el código.
  fetch('/api/buy', { method: 'POST' }); // 👈
}, []);
```

No quieres comprar el producto dos veces. Sin embargo, esta es también la razón por la que no debes poner esta lógica en un Efecto. ¿Qué pasa si el usuario va a otra página y luego presiona atrás? Tu Efecto se ejecutaría de nuevo. No quieres comprar el producto cuando el usuario _visita_ una página; quieres comprarlo cuando el usuario hace _clic_ en el botón de comprar.

Comprar no es causado por un renderizado, sino por una interacción específica. Debería ejecutarse solo cuando el usuario presiona el botón. **Elimina el Efecto y mueve la solicitud `/api/buy` dentro del controlador de evento del botón comprar:**

```jsx
 function handleClick() {
    // ✅ Comprar es un evento porque es causado por una interacción particular.
    fetch('/api/buy', { method: 'POST' }); // 👈
  }
```

**Esto ilustra que si volver a montar rompe la lógica de tu aplicación, esto normalmente descubre errores existentes.** Desde la perspectiva del usuario, visitar una página no debería ser diferente de visitarla, hacer clic en un enlace y, a continuación, pulsar atrás para volver a ver la página. React verifica que tus componentes cumplen este principio volviendo a montarlos una vez en desarrollo.

## ⭐ Poner todo junto

Este playground puede ayudarte a «tener una idea» de cómo funcionan los Efectos en la práctica.

Este ejemplo utiliza [`setTimeout`](https://developer.mozilla.org/es/docs/Web/API/setTimeout) para programar un mensaje en la consola de un elemento input de tipo text que aparecerá tres segundos después de que se ejecute el Efecto. La función de limpieza cancela el tiempo de espera pendiente. Comienza presionando «Montar componente»:

**App.js**
```jsx
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 Programar mensaje"' + text + '"');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancelar mensaje"' + text + '"');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        Mensaje:{' '}
        <input
          value={text}
          onChange={e => setText(e.target.value)}
        />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Desmontar' : 'Montar'} componente
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

Muestra:

![[3-sincronizar-con-efectos-8.png]]

Al pulsar el botón, se monta el componente y aparecen los mensajes `🔵 Programar mensaje "a"`, `🟡 Cancelar mensaje "a"` y `🔵 Programar mensaje "a"` inmediatamente, luego 3 segundos después aparece el mensaje "`⏰ a`":

![[3-sincronizar-con-efectos-9.png]]

Verás tres mensajes al principio: `Programar mensaje "a"`, `Cancelar mensaje "a"`, y `Programar mensaje "a"` de nuevo. Tres segundos después también aparecerá un mensaje que dice `a`. Como aprendiste anteriormente, el programar/cancelar extra se debe a que React vuelve a montar el componente en desarrollo para verificar que has implementado correctamente la limpieza.

Ahora edita el campo de texto para que diga `abc`. Si lo haces lo suficientemente rápido, verás `Programar mensaje "ab"` seguido inmediatamente de `Cancelar mensaje "ab"` y `Programar mensaje "abc"`. **React siempre limpia el Efecto del renderizado anterior antes de ejecutar el Efecto del siguiente renderizado.** Es por eso que incluso si escribes rápidamente en el campo de texto, hay como máximo un temporizador programado a la vez. Edita el campo varias veces y observa la consola para tener una idea de cómo se limpian los Efectos.

Escribe algo dentro del campo e inmediatamente presiona «Desmontar componente». Observa como al desmontar se limpia el Efecto del último renderizado. En este caso, se elimina el último tiempo de espera antes de que tenga la oportunidad de ejecutarse.

Por último, edita el componente de arriba y comenta la función de limpieza para que los tiempos de espera no se cancelen. Intenta escribir `abcde` rápidamente. ¿Qué esperas que suceda en tres segundos? ¿El `console.log(text)` dentro del tiempo de espera imprimirá el `text` _más reciente_ y producirá cinco mensajes `abcde`? ¡Inténtalo para comprobar tu intuición!

Tres segundos después, deberías ver una secuencia de mensajes (`a`, `ab`, `abc`, `abcd`, and `abcde`) en vez de cinco mensajes `abcde`. **Cada Efecto «captura» el valor de `text` desde su correspondiente renderizado.** No importa que el estado `text` haya cambiado: un Efecto del renderizado hecho con `text = 'ab'` siempre verá `'ab'`. En otras palabras, los Efectos de cada renderizado están aislados entre sí. Si tienes curiosidad acerca de cómo funciona esto puedes leer acerca de [clausuras](https://developer.mozilla.org/es/docs/Web/JavaScript/Closures).

### Cada renderizado tiene sus propios Efectos 

**Puedes pensar que `useEffect` como que «añade» un comportamiento a la salida del renderizado**. Considera este Efecto:

```jsx
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>¡Bienvenido a {roomId}!</h1>;
}
```

Veamos qué es lo que sucede exactamente a medida que el usuario navega por la aplicación.

#### Renderizado inicial 

El usuario visita `<ChatRoom roomId="general" />`. Vamos a [sustituir mentalmente](https://es.react.dev/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `roomId` con `'general'`:

```jsx
  // JSX para el primer renderizado (roomId = "general")
  return <h1>¡Bienvenido a general!</h1>;
```

**El Efecto _también_ es una parte de la salida del renderizado.** El Efecto del primer renderizado se convierte en:

```jsx
  // Efecto para el primer renderizado (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencias del primer renderizado (roomId = "general")
  ['general']
```

React ejecuta este Efecto, que se conecta a la sala de chat `'general'`.

#### Volver a renderizar con las mismas dependencias 

Supongamos que `<ChatRoom roomId="general" />` se vuelve a renderizar. La salida JSX es la misma:

```jsx
  // JSX para el segundo renderizado (roomId = "general")
  return <h1>¡Bienvenido a general!</h1>;
```

React observa que la salida del renderizado no ha cambiado, entonces no actualiza el DOM.

El Efecto del segundo renderizado se ve así:

```jsx
  // Efecto del segundo renderizado (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencias para el segundo renderizado (roomId = "general")
  ['general']

```

React compara `['general']` del segundo renderizado con `['general']` del primer renderizado. **Como todas las dependencias son iguales, React _ignora_ el Efecto del segundo renderizado.** Nunca es llamado.

#### Volver a renderizar con diferentes dependencias 

Luego, el usuario visita `<ChatRoom roomId="viaje" />`. Esta vez, el componente devuelve un JSX diferente:

```jsx
  // JSX para el tercer renderizado (roomId = "viaje")
  return <h1>¡Bienvenido a viaje!</h1>;
```

React actualiza el DOM para cambiar `"Bienvenido a general"` a `"Bienvenido a viaje"`.

El Efecto del tercer renderizado se ve así:

```jsx
  // Efecto del tercer renderizado (roomId = "viaje")
  () => {
    const connection = createConnection('viaje');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencias para el tercer renderizado (roomId = "viaje")
  ['viaje']
```

React compara `['viaje']` del tercer renderizado con `['general']` del segundo renderizado. Una dependencia es diferente: `Object.is('viaje', 'general')` es `false`. El Efecto no puede ser omitido.

**Antes de que React pueda aplicar el Efecto de la tercera renderización, necesita limpiar el último Efecto que _sí_ se ejecutó.** El Efecto de la segunda renderización fue omitido, por lo que React necesita limpiar el Efecto de la primer renderización. Si te desplazas hacia arriba hasta la primera renderización, verás que su limpieza llama a `disconnect()` en la conexión que se creó con `createConnection('general')`. Esto desconecta la aplicación de la sala de chat `'general'`.

Después de eso, React ejecuta el Efecto del tercer renderizado que conecta a la sala de chat `'viaje'`.

#### Desmontar

Finalmente, supongamos que el usuario cambia de página y el componente `ChatRoom` se desmonta. React ejecuta la función de limpieza del último Efecto. El último Efecto fue del tercer renderizado. La limpieza del tercer renderizado destruye la conexión a `createConnection('viaje')`. Por lo tanto la aplicación se desconecta de la sala de chat `'viaje'`.

#### Comportamiento solo en modo de desarrollo

Cuando el [Modo Estricto](https://es.react.dev/reference/react/StrictMode) está activado, React vuelve a montar cada componente después de montarlo por primera vez (el estado y DOM se conservan). Esto [te ayuda a encontrar Efectos que necesitan limpieza](https://es.react.dev/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed) y expone tempranamente errores como las condiciones de carrera. Adicionalmente, React vuelve a montar los Efectos cada vez que guardas el archivo en desarrollo. Ambos comportamientos solo suceden en modo de desarrollo.

## Recapitulación

- A diferencia de los eventos, los Efectos son causados por el renderizado en sí mismo en vez de una interacción en particular.
- Los Efectos te permiten sincronizar un componente con sistemas externos (API de terceros, redes, etc).
- Por defecto, los Efectos se ejecutan después de cada renderizado (incluyendo el inicial).
- React omitirá el Efecto si todas sus dependencias tienen los mismos valores que en el anterior renderizado.
- No puedes «elegir» tus dependencias. Son determinadas por el código dentro del Efecto.
- Un array de dependencias vacío (`[]`) corresponde al «montaje» del componente, es decir, cuando se añade en la pantalla.
- En el Modo Estricto, React monta dos veces los componentes (¡Solo en modo de desarrollo!) para poner a prueba tus Efectos.
- Si tu Efecto se rompe debido a que se vuelve a montar, debes implementar una función de limpieza.
- React llamará a tu función de limpieza antes de que el Efecto se ejecute la próxima vez, y también durante el desmontaje.