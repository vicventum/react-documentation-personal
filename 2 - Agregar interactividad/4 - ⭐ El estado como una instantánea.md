Las variables de estado pueden parecerse a las variables normales de JavaScript en las que se puede leer y escribir. Sin embargo, **el estado se comporta más como una instantánea. Al asignarlo no se cambia la variable de estado que ya tienes, sino que se desencadena una _rerenderizado_**.

### Aprenderás

- Cómo la asignación del estado desencadena los rerenderizados.
- Cuándo y cómo se actualiza el estado.
- Por qué el estado no se actualiza inmediatamente después de asignarlo.
- Cómo los controladores de eventos acceden a una «instantánea» del estado.

## ⭐ La asignación de estado desencadena renderizados

Puede que tengas **la idea de tu interfaz de usuario como una que cambia directamente al evento del usuario como un clic**. **En React, funciona un poco diferente de este modelo mental**. 

En la lección anterior, viste que [al asignar estado se solicita un rerenderizado](https://es.react.dev/learn/render-and-commit#step-1-trigger-a-render) de React. Esto significa que para que una interfaz reaccione al evento, es necesario _actualizar el estado_.

En este ejemplo, al pulsar «Enviar», `setIsSent(true)` indica a React que vuelva a renderizar la UI:

**App.jsx**
```jsx
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('¡Hola!');
  if (isSent) {
    return <h1>¡Tu mensaje está en camino!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Mensaje"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Enviar</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```
Muestra: 

![[4-el-estado-como-una-instantanea-1.png]]


Esto es lo que ocurre cuando se hace clic en el botón:

1. Se ejecuta el controlador de evento `onSubmit`.
2. `setIsSent(true)` asigna `isSent` a `true` y **pone en cola un nuevo renderizado**, primero termina de ejecutar la siguiente linea de abajo antes de hacerlo.
3. React vuelve a renderizar el componente según el nuevo valor de `isSent`.

Veamos con más detalle la relación entre estado y renderizado.

## 🌟 El renderizado toma una instantánea en el tiempo 

[«Renderizado»](https://es.react.dev/learn/render-and-commit#step-2-react-renders-your-components) significa que React está llamando a tu componente, que es una función. El JSX que devuelves de esa función es como una instantánea de la UI en el tiempo. Tus props, controladores de eventos y variables locales fueron todos calculados **usando su estado en el momento del renderizado.**

A diferencia de una fotografía o un fotograma de una película, la «_instantánea_» de la interfaz de usuario que devuelves es interactiva. Incluye lógica como controladores de eventos que especifican lo que sucede en respuesta a las entradas. React entonces actualiza la pantalla para que coincida con esta instantánea y conecta los controladores de eventos. Como resultado, al pulsar un botón se activará el controlador de clic de tu JSX.

Cuando React vuelve a renderizar un componente:

1. React vuelve a llamar a tu función.
2. Tu función devuelve una nueva instantánea JSX.
3. React entonces actualiza la pantalla para que coincida con la instantánea devuelta por tu función.
 
![[4-el-estado-como-una-instantanea-2.png]]
>Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

Como memoria de un componente, **el estado no es como una variable regular que desaparece después de que tu función devuelva un valor. El estado en realidad «vive» en el propio React -como si estuviera en una estantería- fuera de tu función**. 

**Cuando React llama a tu componente, te da una instantánea del estado para ese renderizado en particular**. Tu componente devuelve una instantánea de la interfaz de usuario con un nuevo conjunto de accesorios y controladores de eventos en su JSX, todo calculado **usando los valores de estado de ese renderizado**.

![[4-el-estado-como-una-instantanea-3.png]]
>Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

He aquí un pequeño experimento para mostrarte cómo funciona esto. En este ejemplo, se podría esperar que al hacer clic en el botón «+3» se incrementara el contador tres veces porque se llama a `setNumber(number + 1)` tres veces.

Mira lo que ocurre cuando haces clic en el botón «+3»:

**App.js**
```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

Muestra:

![[4-el-estado-como-una-instantanea-4.png]]

Al presionar el botón muestra:

![[4-el-estado-como-una-instantanea-5.png]]

Observa que `number` sólo se incrementa una vez por clic.

**La asignación del estado sólo lo cambia para el _siguiente_ renderizado.** Durante el primer renderizado, `number` era `0`. **Es por eso que en el controlador `onClick` de _ese renderizado_ el valor de `number` sigue siendo `0`, incluso después de que se llamara `setNumber(number + 1)`**:

```jsx
<button onClick={() => {
  setNumber(number + 1);
  setNumber(number + 1);
  setNumber(number + 1);
}}>+3</button>
```

Esto es lo que el controlador de clic de este botón le dice a React que haga:

1. `setNumber(number + 1)`: `number` es `0` así que `setNumber(0 + 1)`.
    - React se prepara para el cambiar `number` a `1` en el siguiente renderizado.
2. `setNumber(number + 1)`: `number` es `0` así que `setNumber(0 + 1)`.
    - React se prepara para el cambiar `number` a `1` en el siguiente renderizado.
3. `setNumber(number + 1)`: `number` es `0` así que `setNumber(0 + 1)`.
    - React se prepara para el cambiar `number` a `1` en el siguiente renderizado.

Aunque hayas llamado a `setNumber(number + 1)` tres veces, en el controlador de evento de _ese renderizado_ `number` es siempre `0`, por lo que asignas el estado a `1` tres veces. Por eso, una vez que el controlador de evento termina, React vuelve a renderizar el componente con `number` igual a `1` en lugar de `3`.

También puedes visualizarlo sustituyendo mentalmente las variables de estado por sus valores en tu código. Haciendo que la variable de estado `number` sea `0` para _ese renderizado_, tu controlador de evento se ve así:

```jsx
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>+3</button>
```

Para el siguiente renderizado, `number` es `1`, así que en _ese renderizado_ el controlador de clics luce así:

```jsx
<button onClick={() => {
  setNumber(1 + 1);
  setNumber(1 + 1);
  setNumber(1 + 1);
}}>+3</button>
```

Por eso, al pulsar de nuevo el botón, el contador se pone en `2`, y luego a `3` en el siguiente clic, y así sucesivamente.

## ⭐ El estado a través del tiempo

Bueno, eso fue divertido. Intenta adivinar que mostrará la alerta al hacer clic en este botón:

**App.js**
```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        alert(number);
      }}>+5</button>
    </>
  )
}
```

Muestra:

![[4-el-estado-como-una-instantanea-6.png]]

Al hacer click, muestra el siguiente alert:

![[4-el-estado-como-una-instantanea-7.png]]

Y al darle en _Aceptar_, se actuliza la pantalla de mostrando:

![[4-el-estado-como-una-instantanea-8.png]]

Si utilizas el método de sustitución de antes, puedes adivinar que la alerta mostrará «0»:

```jsx
setNumber(0 + 5);
alert(0);
```

¿Pero, qué pasa si pones un temporizador en la alerta, de modo que sólo se dispare _después_ de que el componente se vuelva a renderizar? ¿Diría «0» o «5»? Adivínalo.

**App.js**
```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setTimeout(() => {
          alert(number);
        }, 3000);
      }}>+5</button>
    </>
  )
}
```

Muestra:

![[4-el-estado-como-una-instantanea-6.png]]

Al pulsar el botón, se actualiza la pantalla:

![[4-el-estado-como-una-instantanea-8.png]]

Y 5 segundos después, se muestra la alerta con el valor de `0`:

![[4-el-estado-como-una-instantanea-7.png]]

¿Sorprendido? Si se utiliza el método de sustitución, se puede ver la «instantánea» del estado que se pasa a la alerta.

```jsx
setNumber(0 + 5);
setTimeout(() => {  alert(0);}, 3000);
```

El estado almacenado en React puede haber cambiado en el momento en que se ejecuta la alerta, pero se programó utilizando una instantánea del estado en el momento en que el usuario interactuó con ella.

**El valor de una variable de estado nunca cambia dentro de un renderizado,** **incluso si el código de tu controlador de evento es asíncrono**. Dentro del `onClick` de _ese renderizado_, el valor de `number` sigue siendo `0` incluso después de que se llama a `setNumber(number + 5)`. Su valor se «fijó» cuando React «tomó la instantánea» de la UI al llamar a tu componente.

### Otro ejemplo

Aquí hay un ejemplo de cómo eso hace que tus controladores de eventos sean menos propensos a errores de sincronización. A continuación se muestra un formulario que envía un mensaje con un retraso de cinco segundos. Imagina este escenario:

1. Pulsas el botón «Enviar», enviando «Hola» a Alice.
2. Antes de que termine la demora de cinco segundos, cambia el valor del campo «Para» a «Bob».

¿Qué esperas que muestre la alerta (`alert`)? ¿Mostrará «Has dicho Hola a Alice»? ¿O será «Has dicho Hola a Bob»? Haz una suposición con base en lo que sabes y luego pruébalo:

**App.js**
```jsx
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Hola');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`Has dicho ${message} a ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Para:{' '}
        <select
          value={to}
          onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Mensaje"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Enviar</button>
    </form>
  );
}
```

Muestra:

![[4-el-estado-como-una-instantanea-9.png]]

Al pulsar _Enviar_, y cambiar rápidamente el `select` a `Bob`, se actualiza la UI:

![[4-el-estado-como-una-instantanea-10.png]]

Pero 5 segundos después se muestra la alerta pero con el valor de `Alice`:

![[4-el-estado-como-una-instantanea-11.png]]

**React mantiene los valores de estado «fijados» dentro de los controladores de eventos de un renderizado.** No hay que preocuparse de si el estado ha cambiado mientras se ejecuta el código.

Pero, ¿y si quieres leer el último estado antes de un nuevo renderizado? Necesitarás usar una [función de actualización de estado](https://es.react.dev/learn/queueing-a-series-of-state-updates), ¡tratada en la siguiente página!

## Recapitulación

- Asignar un estado solicita un rerenderizado
- React almacena el estado fuera de tu componente, como si estuviera en una estantería.
- Cuando llamas a `useState`, React te da una instantánea del estado _para ese renderizado_.
- Las variables y los controladores de eventos no «sobreviven» a los rerenderizados. Cada renderizado tiene sus propios controladores de eventos.
- Cada renderizado (y las funciones dentro de él) siempre «verán» la instantánea del estado que React dio a _ese_ renderizado.
- Puedes sustituir mentalmente el estado en los controladores de eventos, de forma similar a como piensas en el JSX renderizado.
- Los controladores de eventos creados en el pasado tienen los valores de estado del renderizado en el que fueron creados.