**Cuando quieres que un componente «_recuerde_» alguna información, pero no quieres que esa información [provoque nuevos renderizados](https://es.react.dev/learn/render-and-commit), puedes usar una _ref_**.

### Aprenderás

- Cómo añadir una ref a tu componente
- Cómo actualizar el valor de una ref
- En qué se diferencian las refs y el estado
- Cómo usar las refs de manera segura

## ⭐ Añadir una ref a tu componente 

Puedes añadir una ref a tu componente **importando el Hook `useRef` desde React**:

```jsx
import { useRef } from 'react';
```

Dentro de tu componente, **llama al Hook `useRef` y pasa el valor inicial al que quieres hacer referencia como único argumento**. Por ejemplo, esta es una ref al valor `0`:

```jsx
const ref = useRef(0);
```

`useRef` devuelve un objeto como este:

```jsx
{   current: 0 // El valor que le pasaste a useRef}
```

![[1-referenciar-valores-con-refs-1.png|300]]
> Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

**Puedes acceder al valor actual de esa ref a través de la propiedad `ref.current`. Este valor es mutable intencionalmente, lo que significa que puedes tanto leerlo como modificarlo**. Es como un bolsillo secreto de tu componente que **React no puede rastrear. (Esto es lo que lo hace una «_puerta de escape_» del flujo de datos de una vía de React**: ¡Más sobre eso a continuación!)

Aquí, un botón incrementará `ref.current` en cada clic:

**App.js**
```jsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0); // 👈

  function handleClick() {
    ref.current = ref.current + 1; // 👈
    alert('Hiciste clic ' + ref.current + ' veces!');
  }

  return (
    <button onClick={handleClick}>
      ¡Hazme clic!
    </button>
  );
}
```

Muestra:

![[1-referenciar-valores-con-refs-2.png]]

Al hacer click:

![[1-referenciar-valores-con-refs-3.png]]

La ref hace referencia a un número, pero, al igual que [el estado](https://es.react.dev/learn/state-a-components-memory), podrías hace referencia a cualquier cosa: un string, un objeto, o incluso una función. **A diferencia del estado, la ref es un objeto plano de JavaScript con la propiedad `current` que puedes leer y modificar**.

⭐ Fíjate como **el componente no se rerenderiza con cada incremento.** Al igual que con el estado, las refs son retenidas por React entre rerenderizados. Sin embargo, **asignar el estado rerenderiza un componente. ¡Cambiar una ref no!**

## Ejemplo: crear un cronómetro

**Puedes combinar las refs y el estado en un solo componente**. Por ejemplo, hagamos un cronómetro que el usuario pueda iniciar y detener al presionar un botón. Para poder mostrar cuánto tiempo ha pasado desde que el usuario pulsó «Iniciar», **necesitarás mantener rastreado cuándo el botón de Iniciar fue presionado y cuál es el tiempo actual. Esta información se usa para el renderizado, así que guárdala en el estado:**

```jsx
const [startTime, setStartTime] = useState(null);
const [now, setNow] = useState(null);
```

Cuando el usuario presione «Iniciar», usarás [`setInterval`](https://developer.mozilla.org/docs/Web/API/setInterval) para poder actualizar el tiempo cada 10 milisegundos:

**App.js**
```jsx
import { useState } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null); // 👈
  const [now, setNow] = useState(null); // 👈

  function handleStart() {
    // Empieza a contar.
    setStartTime(Date.now()); // 👈
    setNow(Date.now()); // 👈

    setInterval(() => {
      // Actualiza el tiempo actual cada 10 milisegundos.
      setNow(Date.now()); // 👈
    }, 10);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Tiempo transcurrido: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Iniciar
      </button>
    </>
  );
}
```

Muestra:

![[1-referenciar-valores-con-refs-4.png]]

Al pulsar en el botón se inicia el cronómetro, si se vuelve a pulsar se vuelve a iniciar desde cero:

![[1-referenciar-valores-con-refs-5.png]]

Cuando se presiona el botón «_Detener_», necesitas cancelar el intervalo existente para que deje de actualizar la variable de estado `now`. Puedes hacerlo con una llamada a [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval), pero necesitas pasarle el identificador del intervalo que fue previamente devuelto por la llamada a `setInterval` cuando el usuario presionó Iniciar. Necesitas guardar el identificador del intervalo en alguna parte. **Como el identificador de un intervalo no se usa para el renderizado, puedes guardarlo en una ref:**

**App.js**
```jsx
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null); // 👈

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current); // 👈
    intervalRef.current = setInterval(() => { // 👈
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current); // 👈
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Tiempo transcurrido: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Iniciar
      </button>
      <button onClick={handleStop}>
        Detener
      </button>
    </>
  );
}
```

Muestra:

![[1-referenciar-valores-con-refs-6.png]]

Al pulsar el botón "Iniciar" se inicia el cronómetro:

![[1-referenciar-valores-con-refs-7.png]]

Si pulsamos en el botón "Detener" se detiene el cronómetro:

![[1-referenciar-valores-con-refs-8.png]]

>[!important]
>Cuando una pieza de información es usada para el renderizado, guárdala en el estado. Cuando una pieza de información solo se necesita en los controladores de eventos y no requiere un rerenderizado, usar una ref quizás sea más eficiente.

## ⭐ Diferencias entre las refs y el estado

Tal vez estés pensando que las refs parecen menos «_estrictas_» que el estado —puedes mutarlos en lugar de siempre tener que utilizar una función asignadora del estado, por ejemplo. Pero **en la mayoría de los casos, querrás usar el estado. Las refs son una «_puerta de escape_» que no necesitarás a menudo**. Esta es la comparación entre el estado y las refs:

| las refs                                                                                           | el estado                                                                                                                                                                         |
| -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `useRef(initialValue)` devuelve <br>`{ current: initialValue }`                                    | `useState(initialValue)` devuelve el valor actual de una variable de estado y una función asignadora del estado ( `[value, setValue]`)                                            |
| **No desencadena un rerenderizado cuando** lo cambias.                                             | **Desencadena un rerenderizado cuando lo cambias**.                                                                                                                               |
| **Mutable**: puedes modificar y actualizar el valor de `current` fuera del proceso de renderizado. | »**Immutable**»: necesitas usar la función asignadora del estado para modificar variables de estado para poner en cola un rerenderizado.                                          |
| No deberías leer (o escribir) el valor de `current` durante el renderizado.                        | Puedes leer el estado en cualquier momento. Sin embargo, cada renderizado tiene su propia [instantánea](https://es.react.dev/learn/state-as-a-snapshot) del estado que no cambia. |

Este es un botón contador que está implementado con el estado:

**App.js**
```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Hiciste clic {count} veces
    </button>
  );
}
```

Muestra:

  ![[1-referenciar-valores-con-refs-9.png]]
Al pulsar sobre el botón:

![[1-referenciar-valores-con-refs-10.png]]

**==Como el valor de `count` es mostrado, tiene sentido usar un valor del estado==**. Cuando se asigna el valor del contador con `setCount()`, React rerenderiza el componente y la pantalla se actualiza para reflejar el nuevo contador.

**Si intentaras implementarlo con una ref, pese a que la ref se actualiza, React nunca rerenderizaría el componente, ¡y nunca verías cambiar el contador!** Observa como al hacer clic en este botón **no se actualiza su texto**:

**App.js**
```jsx
import { useRef } from 'react';

export default function Counter() {
  let countRef = useRef(0);

  function handleClick() {
    // ¡Esto no rerenderiza el componente!
    countRef.current = countRef.current + 1;
  }

  return (
    <button onClick={handleClick}>
      Hiciste clic {countRef.current} veces
    </button>
  );
}
```

Muestra:

  ![[1-referenciar-valores-con-refs-9.png]]
Al pulsar sobre el botón no cambia visualmente nada (aunque si lo hace la ref):

![[1-referenciar-valores-con-refs-9.png]]

Esta es la razón por la que leer `ref.current` durante el renderizado conduce a un código poco fiable. Si eso es lo que necesitas, usa en su lugar el estado.


#### ¿Cómo useRef funciona internamente?

**A pesar de que React proporciona tanto `useState` como `useRef`, en principio `useRef` se podría implementar _a partir de_ `useState`**. Puedes imaginar que internamente en React, `useRef` se implementa de esta manera:

```jsx
// Internamente en React
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

**Durante el primer renderizado, `useRef` devuelve `{ current: initialValue }`. React almacena este objeto, así que durante el siguiente renderizado se devolverá el mismo objeto**. Fíjate como el asignador de estado no se usa en este ejemplo. ¡Es innecesario porque `useRef` siempre necesita devolver el mismo objeto!

React proporciona una versión integrada de `useRef` porque es suficientemente común en la practica. Pero **puedes imaginártelo como si fuera una variable de estado normal sin un asignador**. Si estas familiarizado con la programación orientada a objetos, las refs puede que te recuerden a los campos de instancias, pero en lugar de `this.something` escribes `somethingRef.current`.

## ⭐ ¿Cuándo usar refs?

**Típicamente, usarás una ref cuando tu componente necesite «_salir_» de React y comunicarse con APIs externas** —a menudo una API del navegador no impactará en la apariencia de un componente. Estas son algunas de estas situaciones raras:

- Almacenar [identificadores de timeouts](https://developer.mozilla.org/docs/Web/API/setTimeout)
- Almacenar y manipular [elementos del DOM](https://developer.mozilla.org/docs/Web/API/Element), que cubrimos en [la siguiente página](https://es.react.dev/learn/manipulating-the-dom-with-refs)
- Almacenar otros objetos que no son necesarios para calcular el JSX.

Si tu componente necesita almacenar algún valor, pero no impacta la lógica del renderizado, usa refs.

## Buenas prácticas para las refs 

Seguir estos principios hará que tus componentes sean más predecibles:

- **Trata a las refs como una puerta de escape.** Las refs son útiles cuando trabajas con sistemas externos o APIs del navegador. Si mucho de la lógica de tu aplicación y del flujo de los datos depende de las refs, es posible que quieras reconsiderar tu enfoque.
  
- **No leas o escribas `ref.current` durante el renderizado.** Si se necesita alguna información durante el renderizado, usa en su lugar [el estado](https://es.react.dev/learn/state-a-components-memory). Como React no sabe cuándo `ref.current` cambia, incluso leerlo mientras se renderiza hace que el comportamiento de tu componente sea difícil de predecir. (La única excepción a esto es código como `if (!ref.current) ref.current = new Thing()` que solo asigna la ref una vez durante el renderizado inicial).

Las limitaciones del estado en React no se aplican a las refs. Por ejemplo, el estado actúa como una [instantánea para cada renderizado](https://es.react.dev/learn/state-as-a-snapshot) y [no se actualiza de manera síncrona.](https://es.react.dev/learn/queueing-a-series-of-state-updates) Pero cuando mutas el valor actual de una ref, cambia inmediatamente:

```jsx
ref.current = 5;
console.log(ref.current); // 5
```

Esto es porque **la propia ref es un objeto normal de JavaScript,** así que se comporta como uno.

**Tampoco tienes que preocuparte por [evitar la mutación](https://es.react.dev/learn/updating-objects-in-state) cuando trabajas con una ref. Siempre y cuando el objeto que estás mutando no se esté usando para el renderizado, a React no le importa lo que hagas con la ref o con su contenido**.

## Las refs y el DOM

Puedes apuntar una ref a cualquier valor. Sin embargo, **el uso más común para una ref es acceder a un elemento DOM. Por ejemplo, esto es útil si deseas enfocar un input programáticamente**. Cuando pasas una ref a un atributo `ref` en JSX, como `<div ref={myRef}>`, React pondrá el elemento DOM correspondiente en `myRef.current`. Una vez que el elemento es eliminado del DOM, React actualizará `myRef.current` a `null`. Puedes leer más sobre esto en [Manipular el DOM con Refs.](https://es.react.dev/learn/manipulating-the-dom-with-refs)

## Recapitulación

- Las refs son una puerta de escape para guardar valores que no se usan en el renderizado. No las necesitarás a menudo.
- Una ref es un objeto plano de JavaScript con una sola propiedad llamada `current`, que puedes leer o asignarle un valor.
- Puedes pedirle a React que te dé una ref llamando al Hook `useRef`.
- Como el estado, las refs retienen información entre los rerenderizados de un componente.
- A diferencia del estado, asignar el valor de `current` de una ref no desencadena un rerenderizado.
- No leas o escribas `ref.current` durante el renderizado. Esto hace que tu componente sea difícil de predecir.