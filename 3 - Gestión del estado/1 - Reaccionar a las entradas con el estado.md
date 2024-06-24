**React utiliza una forma declarativa para manipular la UI**. **En vez de manipular trozos de la UI de forma individual directamente, describes los diferentes estados en los que puede estar tu componente, y cambias entre ellos en respuesta al lo que haga el usuario**. Esto es similar a como los diseñadores piensan en la UI

### Aprenderás

- Como la programación de UI declarativa se diferencia de la programación de UI imperativa
- Como enumerar los diferentes estados visuales en los que tus componentes pueden estar
- Como forzar los cambios entre los distintos estados desde el código

## Cómo la UI declarativa se compara a la imperativa

Cuando diseñas interacciones con la UI, seguramente pensarás en como la UI _cambia_ en respuesta a las acciones del usuario. Imagina un formulario que permita al usuario enviar una respuesta:

- Cuando escribes algo en el formulario, el botón «_Enviar_» **se habilita.**
- Cuando presionas «_Enviar_», tanto el formulario como el botón **se deshabilitan,** y un indicativo de carga **aparece.**
- Si la petición es exitosa, el formulario **se oculta,** y un mensaje «_Gracias_» **aparece.**
- Si la petición falla, un mensaje de error **aparece,** y el formulario **se habilita** de nuevo.

**En la _programación imperativa_, lo descrito arriba se corresponde directamente con como implementas la interacción. Tienes que escribir las instrucciones exactas para manipular la UI dependiendo de lo que acabe de suceder**. Esta es otra manera de abordar este concepto: imagina acompañar a alguien en un coche mientras le dices paso a paso que tiene que hacer.

![[1-reaccionar-a-las-entradas-con-el-estado-1.png|600]]
> Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

**No sabe a donde quieres ir, solo sigue tus indicaciones. (Y si le das las indicaciones incorrectas, ¡acabarás en el lugar equivocado!) Se llama _imperativo_ por que tienes que «_mandar_» a cada elemento**, desde el indicativo de carga hasta el botón, diciéndole al ordenador _cómo_ tiene que actualizar la UI.

En este ejemplo de UI declarativa, el formulario esta construido _sin_ React. Utiliza el [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) del navegador:

**index.js**
```jsx
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() === 'istanbul') {
        resolve();
      } else {
        reject(new Error('Buen intento, pero incorrecto. ¡Inténtalo de nuevo!'));
      }
    }, 1500);
  });
}

let form = document.getElementById('form');
let textarea = document.getElementById('textarea');
let button = document.getElementById('button');
let loadingMessage = document.getElementById('loading');
let errorMessage = document.getElementById('error');
let successMessage = document.getElementById('success');
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```

**index.html**
```jsx
<form id="form">
  <h2>Cuestionario sobre ciudades</h2>
  <p>
    ¿Qué ciudad se ubica entre dos continentes?
  </p>
  <textarea id="textarea"></textarea>
  <br />
  <button id="button" disabled>Enviar</button>
  <p id="loading" style="display: none">Cargando...</p>
  <p id="error" style="display: none; color: red;"></p>
</form>
<h1 id="success" style="display: none">¡Correcto!</h1>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
</style>
```
Muestra:

![[1-reaccionar-a-las-entradas-con-el-estado-2.png]]

**Manipular la UI de forma imperativa funciona lo suficientemente bien en ejemplos aislados, pero se vuelve mas complicado de manejar de forma exponencial en sistemas complejos**. Imagina actualizar una pagina llena de formularios diferentes como este. Añadir un elemento nuevo a la UI o una nueva interacción requeriría revisar todo el código existente meticulosamente para asegurarse de no haber introducido un bug (por ejemplo, olvidando mostrar u ocultar algo).

React fue construido para solucionar este problema.

**En React, no necesitas manipular directamente la UI,lo que significa que no necesitas habilitar, deshabilitar, mostrar, u ocultar los componentes directamente. En su lugar, tú _declaras lo que quieres mostrar_, y React averigua cómo actualizar la UI**. 

Piensa en ello como montarte en un taxi y decirle al conductor a donde quieres ir en lugar de decirle paso a paso qué hacer. Es el trabajo del conductor llevarte a tu destino, ¡e incluso conocerá algún atajo que no habías considerado!

![[1-reaccionar-a-las-entradas-con-el-estado-3.png]]
>Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

## Pensar en la UI de forma declarativa 

Arriba has visto como implementar un formulario de forma imperativa. Para entender mejor como pensar en React, recorrerás el ejemplo reimplementando esta UI en React más abajo:

1. **Identifica** los diferentes estados visuales de tu componente
2. **Determina** qué produce esos cambios de estado
3. **Representa** el estado en memoria usando `useState`
4. **Elimina** cualquier variable de estado no esencial
5. **Conecta** los controladores de eventos para actualizar el estado

### Paso 1: Identifica los diferentes estados visuales de tu componente - Maquetar todos los posibles estados

En las ciencias de la computación, tal vez escucharás algo sobre una [«_máquina de estado_»](https://en.wikipedia.org/wiki/Finite-state_machine) siendo este uno de muchos «_estados_». Si trabajas con un diseñador, habrás visto bocetos para diferentes «_estados visuales_». React se encuentra en un punto intermedio de diseño y ciencias de la computación, asi que ambas ideas son fuentes de inspiración.

Primero, necesitas visualizar todos los diferentes «_estados_» de la UI que el usuario pueda ver:

- **Vacío**: El formulario tiene deshabilitado el botón «_Enviar_».
- **Escribiendo**: El formulario tiene habilitado el botón «_Enviar_».
- **Enviando**: El formulario está completamente deshabilitado. Se muestra un indicador de carga.
- **Éxito**: El mensaje «_Gracias_» se muestra en lugar del formulario.
- **Error**: Igual que el estado de Escribiendo, pero con un mensaje de error extra.

Al igual que un diseñador, querrás «_esbozar_» o crear «_bocetos_» para los diferentes estados antes de añadir tu lógica. Por ejemplo, aquí hay un boceto solo para la parte visual del formulario. Este boceto es controlado por una prop llamado `status` con valor por defecto de `'empty'`:

**App.js**
```jsx
export default function Form({
  status = 'empty'
}) {
  if (status === 'success') {
    return <h1>¡Correcto!</h1>
  }
  return (
    <>
      <h2>Cuestionario sobre ciudades</h2>
      <p>
       ¿En qué ciudad hay un cartel que convierte el aire en agua potable?
      </p>
      <form>
        <textarea />
        <br />
        <button>
          Enviar
        </button>
      </form>
    </>
  )
}
```

Muestra:

![[1-reaccionar-a-las-entradas-con-el-estado-4.png]]

Podrías llamar a ese prop de la forma que quisieras, el nombre no es importante. Prueba a editar `status = 'empty'` a `status = 'success'` para que veas el mensaje aparecer. **Esbozar te permita iterar en la UI rápidamente antes de comenzar con la lógica**. Aquí hay una versión algo más desarrollada del mismo componente, todavía «_controlada_» por la prop `status`:

**App.js**
```jsx
export default function Form({
  // Try 'submitting', 'error', 'success':
  status = 'empty'
}) {
  if (status === 'success') {
    return <h1>¡Correcto!</h1>
  }
  return (
    <>
      <h2>Cuestionario sobre ciudades</h2>
      <p>
        ¿En qué ciudad hay un cartel que convierte el aire en agua potable?
      </p>
      <form>
        <textarea disabled={
          status === 'submitting'
        } />
        <br />
        <button disabled={
          status === 'empty' ||
          status === 'submitting'
        }>
          Enviar
        </button>
        {status === 'error' &&
          <p className="Error">
            Buen intento, pero incorrecto. ¡Intntalo de nuevo!
          </p>
        }
      </form>
      </>
  );
}
```

Muestra:

![[1-reaccionar-a-las-entradas-con-el-estado-4.png]]

#### ⭐ Mostrar muchos estados visuales a la vez

Si un componente tiene un montón de estados visuales, puede resultar conveniente mostrarlos todos en una página:

**App.js**
```jsx
import Form from './Form.js';

let statuses = [
  'empty',
  'typing',
  'submitting',
  'success',
  'error',
];

export default function App() {
  return (
    <>
      {statuses.map(status => (
        <section key={status}>
          <h4>Form ({status}):</h4>
          <Form status={status} />
        </section>
      ))}
    </>
  );
}
```

**Form.js**
```jsx
export default function Form({ status }) {
  if (status === 'success') {
    return <h1>¡Correcto!</h1>
  }
  return (
    <form>
      <textarea disabled={
        status === 'submitting'
      } />
      <br />
      <button disabled={
        status === 'empty' ||
        status === 'submitting'
      }>
        Enviar
      </button>
      {status === 'error' &&
        <p className="Error">
          Buen intento, pero incorrecto. ¡Inténtalo de nuevo!
        </p>
      }
    </form>
  );
}
```

![[1-reaccionar-a-las-entradas-con-el-estado-5.png]]

Páginas como estas son comúnmente llamadas como «_guías de estilo en vivo_» o «_storybooks_».

### Paso 2: Determina qué produce esos cambios de estado 

Puedes desencadenar actualizaciones de estado en respuesta a dos tipos de entradas:

- **Entradas humanas,** como hacer click en un botón, escribir en un campo, navegar a un link.
- **Entradas del ordenador,** como recibir una respuesta del navegador, que se complete un _timeout_, una imagen cargando.

![[1-reaccionar-a-las-entradas-con-el-estado-6.png]]
> Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

En ambos casos, **debes declarar [variables de estado](https://es.react.dev/learn/state-a-components-memory#anatomy-of-usestate) para actualizar la UI.** Para el formulario que vas a desarrollar, necesitarás cambiar el estado en respuesta de diferentes entradas:

- **Cambiar la entrada de texto** (humano) debería cambiar del estado _Vacío_ al estado _Escribiendo_ o al revés, dependiendo de si la caja de texto está vacía o no.
- **Hacer click el el botón Enviar** (humano) debería cambiarlo al estado _Enviando_ .
- **Una respuesta exitosa de red** (ordenador) debería cambiarlo al estado _Éxito_.
- **Una respuesta fallida de red** (ordenador) debería cambiarlo al estado _Error_ con el mensaje de error correspondiente.

> [!note]
> ¡Ten en cuenta que las entradas humanas suelen necesitar [controladores de eventos](https://es.react.dev/learn/responding-to-events)!

Para ayudarte a visualizar este flujo, intenta dibujar cada estado en papel como un círculo etiquetado, y cada cambio entre dos estados como una flecha. Puedes esbozar muchos flujos de esta manera y ordenar los errores mucho antes de la implementación:


![[1-reaccionar-a-las-entradas-con-el-estado-7.webp|600]]
>Estados del formulario

### Paso 3: Representa el estado en memoria usando `useState`

A continuación, necesitarás representar los estados visuales de tu componente en la memoria con [`useState`.](https://es.react.dev/reference/react/useState) La simplicidad es clave: cada pieza de estado es una «_pieza en movimiento_», y **quieres tan pocas «_piezas en movimiento_» como sea posible.** ¡Más complejidad conduce a más errores!

Comienza con el estado que _absolutamente debe_ estar allí. Por ejemplo, necesitarás almacenar la `respuesta` para la entrada y el `error` (si existe) para almacenar el último error:

```jsx
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

Después, necesitarás una variable de estado que represente cuál de los estados visuales descritos anteriormente quieres mostrar. Generalmente hay más de una manera de representarlo en la memoria, por lo que necesitarás experimentar con ello.

Si tienes dificultades para pensar en la mejor manera inmediatamente, comienza agregando suficiente estado para que _definitivamente_ estés seguro de que todos los posibles estados visuales están cubiertos:

```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', o 'success'
```

Tu primera idea probablemente no sea la mejor, ¡pero está bien! ¡Refactorizar el estado es parte del proceso!

### ⭐ Paso 4: Elimina cualquier variable de estado no esencial

**Deberías evitar la duplicación en el contenido del estado, por lo que solo rastrearás lo que es esencial**. Dedicar un poco de tiempo a refactorizar su estructura de estado hará que tus componentes sean más fáciles de entender, reducirá la duplicación y evitará significados no deseados. Tu objetivo es **prevenir los casos en los que el estado en la memoria no represente ninguna UI válida que te gustaría que viera un usuario.** (Por ejemplo, nunca deberías mostrar un mensaje de error y deshabilitar la entrada al mismo tiempo, ¡o el usuario no podría corregir el error!)

Aquí hay algunas preguntas que podrías hacerte sobre tus variables de estado:

- **¿Significa que el estado causa un paradoja?** Por ejemplo, `isTyping` y `isSubmitting` no pueden ser ambos `true`. Un paradoja generalmente significa que el estado no está lo suficientemente restringido. Hay cuatro combinaciones posibles de dos booleanos, pero solo tres corresponden a estados válidos. Para eliminar el estado «_imposible_», puede combinarlos en un `status` que debe ser uno de tres valores: `'typing'`, `'submitting'`, o `'success'`.
  
- **¿La misma información está disponible en otra variable de estado ya?** Otra paradoja: `isEmpty` y `isTyping` no pueden ser `true` al mismo tiempo. Al hacerlos variables de estado separadas, corre el riesgo de que se desincronicen y causen errores. Afortunadamente, se puede eliminar `isEmpty` y en su lugar verificar `answer.length === 0`.
  
- ⭐ **¿Se puede obtener la misma información de la inversa de otra variable de estado?** `isError` no es necesario porque se puede comprobar `error !== null` en su lugar.

Después de esta limpieza, nos quedamos con 3 (¡a partir de 7!) variables de estado _esenciales_ :

```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', o 'success'
```

**Sabes que son esenciales, porque no puedes eliminar ninguna de ellos sin romper la funcionalidad**.

#### Eliminar estados «_imposibles_» con un reducer

Estas tres variables son una representación suficientemente buena del estado de este formulario. Sin embargo, todavía hay algunos estados intermedios que no tienen sentido. Por ejemplo, un `error` no nulo no tiene sentido cuando `status` es `'success'`. Para modelar el estado con más precisión, puedes [extraerlo en un reducer.](https://es.react.dev/learn/extracting-state-logic-into-a-reducer) ¡Los reducers le permiten unificar múltiples variables de estado en un solo objeto y consolidar toda la lógica relacionada!

### Paso 5: Conecta los controladores de eventos para actualizar el estado 

Por último, **crea controladores de eventos para establecer las variables de estado**. A continuación se muestra el formulario final, con todos los controladores de eventos conectados:

**App.js**
```jsx
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>¡Correcto!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>Cuestionario sobre ciudades</h2>
      <p>
        ¿En qué ciudad hay un cartel que convierte el aire en agua potable?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Enviar
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Buen intento, pero incorrecto. ¡Inténtalo de nuevo!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

Muestra:

![[1-reaccionar-a-las-entradas-con-el-estado-4.png]]

Aunque este código es más largo que el ejemplo imperativo original, es mucho menos frágil. Expresar todas las interacciones como cambios de estado te permite introducir nuevos estados visuales sin romper los existentes. También te permite cambiar lo que se debe mostrar en cada estado sin cambiar la lógica de la interacción en sí.

## Recapitulación

- La programación declarativa significa describir la interfaz de usuario para cada estado visual en lugar de microgestionar la interfaz de usuario (imperativa).
- Cuando desarrolles un componente:
    1. Identifica todos sus estados visuales.
    2. Determina los disparadores humanos y de computadora para los cambios de estado.
    3. Modela el estado con `useState`.
    4. Elimina el estado no esencial para evitar errores y paradojas.
    5. Conecta los controladores de eventos para actualizar el estado.