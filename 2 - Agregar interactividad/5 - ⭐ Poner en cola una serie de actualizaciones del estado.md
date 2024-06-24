**Al asignar una variable de estado se pondrá en cola otro renderizado. Pero a veces, es posible que quieras realizar varias operaciones antes de poner en cola el siguiente renderizado. Para hacer esto, nos ayuda entender cómo React realiza las actualizaciones de estado por lotes**.

### Aprenderás

- Qué es «_la actualización por lotes (_batching_)_» y cómo lo utiliza React para procesar múltiples actualizaciones del estado
- Cómo aplicar varias actualizaciones a la misma variable de estado de forma consecutiva

## ⭐ React actualiza el estado por lotes

Podrías esperar que al hacer clic en el botón «_+3_» el contador se incremente tres veces porque llama a `setNumber(number + 1)` tres veces:

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

Al pulsar sobre el botón muestra:
![[4-el-estado-como-una-instantanea-5.png]]
![[4-el-estado-como-una-instantanea-5.png]]


Sin embargo, como se puede recordar de la sección anterior, **[[4 - ⭐ El estado como una instantánea#🌟 El renderizado toma una instantánea en el tiempo|los valores del estado de cada renderizado son fijos]]), por lo que el valor de `number` dentro del controlador de evento del primer renderizado siempre es `0`, sin importar cuántas veces se llame a `setNumber(1)`**:

```jsx
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

Pero hay otro factor a analizar aquí. ==**React espera a que _todo_ el código de los controladores de eventos se haya ejecutado antes de procesar tus actualizaciones de estado.**== **Por ello, el rerenderizado sólo se produce _después_ de todas las llamadas `setNumber()`**.

Esto puede recordarte a un camarero que toma nota de un pedido en un restaurante. ¡El camarero no corre a la cocina al mencionar tu primer plato! En cambio, te deja terminar tu pedido, te permite hacerle cambios e incluso toma nota de los pedidos de las otras personas en la mesa.


![Un elegante cursor en un restaurante hace un pedido varias veces con React, que interpreta el papel de camarero. Después de que ella llama a setState() múltiples veces, el camarero anota lo último que ella pidió como su pedido final.|400](https://es.react.dev/images/docs/illustrations/i_react-batching.png)
>Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

Esto te permite actualizar múltiples variables del estado -incluso de múltiples componentes- sin realizar demasiados [rerenderizados.](https://es.react.dev/learn/render-and-commit#re-renders-when-state-updates) Pero **esto también significa que la UI no se actualizará hasta _después_ de que tu controlador de evento, y cualquier código en él, se complete**. 

**Este comportamiento, también conocido como _batching_, hace que tu aplicación de React se ejecute mucho más rápido**. También evita tener que lidiar con confusos renderizados «_a medio terminar_» en los que sólo se han actualizado algunas de las variables.

**React no agrupa _múltiples_ eventos intencionados como los clics** —cada clic se controla por separado. Puedes estar seguro de que React sólo actualizará por lotes cuando sea seguro hacerlo. Esto garantiza que, por ejemplo, si el primer clic del botón desactiva un formulario, el segundo clic no lo enviará de nuevo.

## 🌟 Actualización de la misma variable de estado varias veces antes del siguiente renderizado

Es un caso de uso poco común, pero si quieres actualizar la misma variable de estado varias veces antes del siguiente renderizado, en lugar de pasar el _siguiente valor de estado_ como `setNumber(number + 1)`, **puedes pasar una _función_ que calcule el siguiente estado basado en el anterior en la cola, como `setNumber(n => n + 1)`. Es una forma de decirle a React que «_haga algo con el valor del estado_» en lugar de simplemente reemplazarlo**.

Intenta incrementar el contador ahora:

**App.js**
```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```


Muestra:

![[4-el-estado-como-una-instantanea-4.png]]

Al pulsar sobre el botón muestra:

![[5-actualizaciones-de-estado-en-cola-1.png]]

Aquí, `n => n + 1` se la llama una **función de actualización.** Cuando la pasas a una asignación de estado:

1. **React pone en cola esta función para que se procese después de que se haya ejecutado el resto del código del controlador de evento**.
2. **Durante el siguiente renderizado, React recorre la cola y te da el estado final actualizado**.

```jsx
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

Así es como funciona React a través de estas líneas de código mientras se ejecuta el controlador de evento:

1. `setNumber(n => n + 1)`: `n => n + 1` es una función. React la añade a la cola.
2. `setNumber(n => n + 1)`: `n => n + 1` es una función. React la añade a la cola.
3. `setNumber(n => n + 1)`: `n => n + 1` es una función. React la añade a la cola.

Cuando llamas a `useState` durante el siguiente renderizado, React recorre la cola. El estado anterior `number` era `0`, así que eso es lo que React pasa a la primera función actualizadora como el argumento `n`. Luego React toma el valor de devolución de tu función actualizadora anterior y lo pasa al siguiente actualizador como `n`, y así sucesivamente:

| Actualización en cola | `n` | Devuelve    |
| --------------------- | --- | ----------- |
| `n => n + 1`          | `0` | `0 + 1 = 1` |
| `n => n + 1`          | `1` | `1 + 1 = 2` |
| `n => n + 1`          | `2` | `2 + 1 = 3` |

React almacena `3` como resultado final y lo devuelve desde `useState`.

Por eso, al hacer clic en «_+3_» en el ejemplo anterior, el valor se incrementa correctamente en 3.

### ⭐ ¿Qué ocurre si se actualiza el estado después de sustituirlo?

¿Qué pasa con este controlador de evento? ¿Qué valor crees que tendrá `number` en el próximo renderizado?

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

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
        setNumber(n => n + 1);
      }}>Incrementa el número</button>
    </>
  )
}
```

Muestra:

![[5-actualizaciones-de-estado-en-cola-2.png]]

Al hacer click muestra:

![[5-actualizaciones-de-estado-en-cola-3.png]]

Esto es lo que este controlador de evento le dice a React que haga:

1. `setNumber(number + 5)`: `number` es `0`, así que `setNumber(0 + 5)`. React añade «_reemplazar con `5`_» a su cola.
2. `setNumber(n => n + 1)`: `n => n + 1` es una función de actualización. React añade _esa función_ a su cola.

Durante el siguiente renderizado, React recorre la cola de estados:

|Actualización en cola|`n`|Devuelve|
|---|---|---|
|_»reemplazar con `5`_»|`0` (sin usar)|`5`|
|`n => n + 1`|`5`|`5 + 1 = 6`|

React almacena `6` como resultado final y lo devuelve desde `useState`.

>[!note]
> Te habrás dado cuenta de que `setState(x)` en realidad funciona como `setState(n => x)`, ¡pero `n` no se utiliza!

### ⭐ ¿Qué ocurre si se sustituye el estado después de actualizarlo?

Probemos un ejemplo más. ¿Qué valor crees que tendrá «_number_» en el próximo renderizado?

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

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
        setNumber(n => n + 1);
        setNumber(42);
      }}>Incrementa el número</button>
    </>
  )
}
```

Muestra:

![[5-actualizaciones-de-estado-en-cola-2.png]]

Muestra:

![[5-actualizaciones-de-estado-en-cola-4.png]]

Así es como funciona React a través de estas líneas de código mientras se ejecuta este controlador de evento:

1. `setNumber(number + 5)`: `number` es `0`, así que `setNumber(0 + 5)`. React añade «_reemplazar con `5`_» a su cola.
2. `setNumber(n => n + 1)`: `n => n + 1` es una función de actualización. React añade _esa función_ a su cola.
3. `setNumber(42)`: React añade «_reemplazar con `42`_» a su cola.

Durante el siguiente renderizado, React recorre la cola de estados:

| Actualización en cola   | `n`            | Devuelve    |
| ----------------------- | -------------- | ----------- |
| «_reemplazar con `5`_»  | `0` (sin usar) | `5`         |
| `n => n + 1`            | `5`            | `5 + 1 = 6` |
| «_reemplazar con `42`_» | `6` (sin usar) | `42`        |

Entonces React almacena `42` como resultado final y lo devuelve desde `useState`.

Para resumir, así es como puedes pensar en lo que estás pasando a la función de asignación de estado `setNumber`:

- **Una función de actualización** (p. ej. `n => n + 1`) se añade a la cola.
- **Cualquier otro valor** (p. ej. pasarle un `5`) añade «_reemplazar con `5`_» a la cola, ignorando lo que ya está en cola.

Después de que el controlador de evento se complete, React lanzará un rerenderizado. Durante el rerenderizado, React procesará la cola. Las funciones de actualización se ejecutan durante el renderizado, por lo que **las funciones de actualización deben ser [puras](https://es.react.dev/learn/keeping-components-pure)** y sólo _devuelven_ el resultado. **No intentes establecer el estado desde dentro de ellas o ejecutar otros efectos secundarios**. En modo estricto, React ejecutará cada función de actualización dos veces (pero descartará el segundo resultado) para ayudarte a encontrar errores.

### Convenciones de nomenclatura

Es habitual nombrar el argumento de la función de actualización por las primeras letras de la variable de estado correspondiente:

```jsx
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

Si prefieres un código más detallado, otra opción habitual es repetir el nombre completo de la variable del estado, como `setEnabled(enabled => !enabled)`, o utilizar un prefijo como `setEnabled(prevEnabled => !prevEnabled)`.

## Recapitulación

- **Establecer el estado no cambia la variable en el renderizado existente, pero si solicita un nuevo renderizado**.
- **React procesa las actualizaciones de estado después de que los controladores de eventos hayan terminado de ejecutarse**. Esto se llama _batching_.
- **Para actualizar algún estado varias veces en un evento, puedes utilizar la función de actualización `setNumber(n => n + 1)`**.