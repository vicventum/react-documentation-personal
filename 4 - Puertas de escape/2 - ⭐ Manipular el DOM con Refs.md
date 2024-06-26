React automáticamente actualiza el [DOM](https://developer.mozilla.org/es/docs/Web/API/Document_Object_Model/Introduction) para que coincida con tu salida de renderizado, por lo que tus componentes no necesitarán manipularlo con frecuencia. Sin embargo, **a veces es posible que necesites acceder a los elementos del DOM gestionados por React**, por ejemplo, enfocar un nodo, desplazarse hasta él, o medir su tamaño y posición. **No hay una forma integrada para hacer ese tipo de cosas en React, por lo que necesitarás una _ref_ al nodo DOM**.

### Aprenderás

- Cómo acceder a un nodo DOM gestionado por React con el atributo `ref`
- Cómo el atributo `ref` de JSX se relaciona con el Hook `useRef`
- Cómo acceder al nodo DOM de otro componente
- En qué casos es seguro modificar el DOM gestionado por React

## Obtener una ref del nodo

Para acceder a un nodo DOM gestionado por React, primero importa el Hook `useRef`:

```jsx
import { useRef } from 'react';
```

Luego, úsalo para declarar una ref dentro de tu componente

```jsx
const myRef = useRef(null);
```

Finalmente, pasa la ref como el atributo `ref` a la etiqueta JSX en el que quieres obtener el nodo DOM:

```jsx
<div ref={myRef}>
```

**El _Hook_ `useRef` devuelve un objeto con una sola propiedad llamada `current`. Inicialmente, `myRef.current` va a ser `null`. Cuando React cree un nodo DOM para este `<div>`, React pondrá una referencia a este nodo en `myRef.current`. Entonces podrás acceder a este nodo DOM desde tus [controladores de eventos](https://es.react.dev/learn/responding-to-events) y usar las [API de navegador](https://developer.mozilla.org/es/docs/Web/API/Element) integradas definidas en él**.

```jsx
// Puedes usar cualquier API de navegador, por ejemplo:
myRef.current.scrollIntoView();
```

### ⭐ Ejemplo: Enfocar el campo de texto input

En este ejemplo, hacer clic en el botón va a enfocar el _input_:

**App.js**
```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null); // 👈

  function handleClick() {
    inputRef.current.focus(); // 👈
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Enfocar el input
      </button>
    </>
  );
}
```

Muestra:

![[2-manipular-el-dom-con-refs-1.png]]

Si pulsamos en el botón vemos cómo ahora el input se enfoca:

![[2-manipular-el-dom-con-refs-2.png]]

Para implementar esto:

1. Declara `inputRef` con el _Hook_ `useRef`.
2. Pásalo como `<input ref={inputRef}>`. Esto le dice a React que **coloque el nodo DOM `<input>` en `inputRef.current`.**
3. En la función `handleClick`, lee el nodo _input_ del DOM desde `inputRef.current` y llama a [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) en él con `inputRef.current.focus()`.
4. Pasa el controlador de evento `handleClick` a `<button>` con `onClick`.

**Mientras manipular el DOM es el caso de uso más común para las refs, el _Hook_ `useRef` puede ser usado para almacenar otras cosas fuera de React, como las ID de temporizadores. De manera similar al estado, las refs permanecen entre renderizados. ==Las refs son como variables de estado que no desencadenan nuevos renderizados cuando las pones==**. Lee acerca de las refs en [Referenciar valores con refs.](https://es.react.dev/learn/referencing-values-with-refs)

### Ejemplo: Desplazarse a un elemento

**Puedes tener más de una _ref_ en un componente**. En este ejemplo, hay un carrusel de tres imágenes. Cada botón centra una imagen al llamar al método del navegador [`scrollIntoView()`](https://developer.mozilla.org/es/docs/Web/API/Element/scrollIntoView) en el nodo DOM correspondiente:

**App.js**
```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const firstCatRef = useRef(null); // 👈
  const secondCatRef = useRef(null); // 👈
  const thirdCatRef = useRef(null); // 👈

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({ // 👈
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({ // 👈
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({ // 👈
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>
          Tom
        </button>
        <button onClick={handleScrollToSecondCat}>
          Maru
        </button>
        <button onClick={handleScrollToThirdCat}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
              ref={firstCatRef} // 👈
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
              ref={secondCatRef} // 👈
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
              ref={thirdCatRef} // 👈
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

Muestra:

![[2-manipular-el-dom-con-refs-3.png]]

Si pulsamos sobre el botón "Maru", se centra la imagen del felino correspondiente a ese nombre:

![[2-manipular-el-dom-con-refs-4.png]]

#### ⭐ Cómo gestionar una lista de refs usando un callback ref

En los ejemplos de arriba, hay un número predefinido de refs. Sin embargo, **algunas veces es posible que necesites una ref en cada elemento de la lista, y no sabes cuantos vas a tener**. Algo como esto **no va a funcionar**:

```jsx
<ul>
  {items.map((item) => {
    // ¡No funciona!
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

Esto es porque **los Hooks solo tienen que ser llamados en el nivel más alto de tu componente**. No puedes llamar a `useRef` en un bucle, en una condición, o dentro de una llamada a `map()`.

- Una posible forma de evitar esto es hacer una sola _ref_ a su elemento padre, y luego usar métodos de manipulación del DOM como [`querySelectorAll`](https://developer.mozilla.org/es/docs/Web/API/Document/querySelectorAll) para «_encontrar_» los nodos hijos individuales a partir de él. Sin embargo, esto es frágil y puede romperse si la estructura del DOM cambia.

- Otra solución es **pasar una función al atributo `ref`.** A esto se le llama **un [callback `ref`.](https://es.react.dev/reference/react-dom/components/common#ref-callback)** **React llamará tu callback ref con el nodo DOM cuando sea el momento de poner la ref, y con `null` cuando sea el momento de limpiarla**. Esto te permite mantener tu propio array o un [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map), y acceder a cualquier ref por su índice o algún tipo de ID.

Este ejemplo te muestra cómo puedes usar este enfoque para desplazarte a un nodo arbitrario en una lista larga:

**App.jsx**
```jsx
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef(null);
  const [catList, setCatList] = useState(setupCatList);

  function scrollToCat(cat) {
    const map = getMap();
    const node = map.get(cat);
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Inicializa el Map en el primer uso
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToCat(catList[0])}>Tom</button>
        <button onClick={() => scrollToCat(catList[5])}>Maru</button>
        <button onClick={() => scrollToCat(catList[9])}>Jellylorum</button>
      </nav>
      <div>
        <ul>
          {catList.map((cat) => (
            <li
              key={cat}
              ref={(node) => { // 👈
                const map = getMap();
                if (node) {
                  map.set(cat, node);
                } else {
                  map.delete(cat);
                }
              }}
            >
              <img src={cat} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catList = [];
  for (let i = 0; i < 10; i++) {
    catList.push("https://loremflickr.com/320/240/cat?lock=" + i);
  }

  return catList;
}
```

En este ejemplo, `itemsRef` no contiene un solo nodo DOM. En su lugar, contiene un [Map](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Map) desde el ID del elemento hasta un nodo DOM. ([¡Las refs pueden contener cualquier valor!](https://es.react.dev/learn/referencing-values-with-refs)) El [callback `ref`](https://es.react.dev/reference/react-dom/components/common#ref-callback) en cada elemento de la lista se encarga de actualizar el Map:

```jsx
<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    if (node) {
      // Add to the Map
      map.set(cat, node);
    } else {
      // Remove from the Map
      map.delete(cat);
    }
  }}
>
```

Esto te permite leer nodos DOM individuales del Map más tarde.>

> [!danger]
>#### Experimental
> Este ejemplo muestra otro enfoque para el manejo del Map con una función callback _cleanup_ de  `ref`
>```jsx
>\<li
>  key={cat.id}
>  ref={node => {
>    const map = getMap();
>    // Add to the Map
>    map.set(cat, node);>
>
>    return () => {
>      // Remove from the Map
>      map.delete(cat);
>    };
>  }}
>```

## ⭐ Accediendo a nodos DOM de otros componentes

> [!important]
> En la próxima versión de React (v19), existirá la posibilidad de enviar una `ref` como props, por lo que este enfoque con `forwardRef` quedará obsoleto

Cuando colocas una _ref_ en un componente integrado que devuelve de salida un elemento del navegador como `<input />`, React establecerá la propiedad `current` de esa _ref_ al nodo DOM correspondiente (como el `<input />` real del navegador)

Sin embargo, **si intentas poner una ref en tu propio componente, como `<MyInput />`, por defecto tendrás `null`**. Aquí hay un ejemplo demostrándolo. Nota como al hacer clic en el botón **no** enfoca el _input_.

**App.js**
```jsx
import { useRef } from 'react';

function MyInput(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} /> // 👈 ❌
      <button onClick={handleClick}>
        Enfocar el input
      </button>
    </>
  );
}
```

Muestra:

![[2-manipular-el-dom-con-refs-1.png]]

Al hacer click el input no se enfoca:

![[2-manipular-el-dom-con-refs-1.png]]

Para ayudarte a notar el problema, React también mostrará un error en la consola.

> [!warning]
>_Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?_

**(Traducción)**

>[!warning]
>_Advertencia: Los componentes de función no pueden recibir refs. Los intentos de acceder a esta ref fallarán. ¿Querías usar React.forwardRef()?_

Esto ocurre porque **por defecto React no permite que un componente acceda a los nodos DOM de otros componentes. ¡Ni siquiera a sus propios hijos!** Esto es intencionado. Las refs son una vía de escape que debe usarse con moderación. Manipular manualmente los nodos DOM de _otro_ componente hace tu código aún más frágil.

En cambio, **los componentes que _quieran_ exponer sus nodos DOM tienen que _optar_ por ese comportamiento explícitamente. Un componente puede especificar que «_reenvíe_» su ref a uno de sus hijos. Aquí vemos como `MyInput` puede usar la API `forwardRef`**:

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

Así es como funciona:

1. `<MyInput ref={inputRef} />` le dice a React que coloque el nodo DOM correspondiente en `inputRef.current`. Sin embargo, depende del componente `MyInput` utilizarlo o no; por defecto no lo hace.
2. El componente `MyInput` es declarado usando `forwardRef`. **Esto hace que pueda optar por recibir el `inputRef` como segundo argumento de `ref` la cual está declarada después de `props`**.
3. `MyInput` por si mismo pasa la `ref` que recibió del `<input>` dentro de él.

Ahora al hacer clic en el botón para enfocar el _input_, funciona:

**App.js**
```jsx
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => { // 👈 3
  return <input {...props} ref={ref} />; // 👈 4
});

export default function Form() {
  const inputRef = useRef(null); // 👈 1

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} /> // 👈 2
      <button onClick={handleClick}>
        Enfocar el input
      </button>
    </>
  );
}
```

> [!tip]
>En diseño de sistemas, es un patrón común para componentes de bajo nivel como botones, _inputs_, etc. reenviar sus refs a sus nodos DOM. Por otro lado, los componentes de alto nivel como formularios, listas, o secciones de página, usualmente no suelen exponer sus nodos DOM para evitar dependencias accidentales de la estructura del DOM.

#### ⭐ Exponiendo un subconjunto de la API con un manejador imperativo

**En el ejemplo de arriba, `MyInput` expone el elemento _input_ del DOM original. Esto le permite al componente padre llamar a `focus()` en él. Sin embargo, esto también le permite al componente padre hacer otra cosa**, por ejemplo, cambiar sus estilos CSS. En casos pocos comunes, **quizás quieras restringir la funcionalidad expuesta. Puedes hacer eso con `useImperativeHandle`**:

**App.js**
```jsx
import {
  forwardRef, 
  useRef, 
  useImperativeHandle
} from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({ // 👈
    // Solo expone focus y nada más
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Enfocar el input
      </button>
    </>
  );
}
```

Aquí, `realInputRef` dentro de `MyInput` mantiene el nodo DOM de input actual. Sin embargo, **`useImperativeHandle` indica a React a proveer tu propio objeto especial como el valor de una ref al componente padre. Por lo tanto, `inputRef.current` dentro del componente `Form` solo va a tener el método `focus`**. En este caso, el «manejador» ref no es el nodo DOM, sino el objeto personalizado que creaste dentro de la llamada de `useImperativeHandle`.

## Cuando React adjunta las refs

En React, cada actualización está dividida en [dos fases](https://es.react.dev/learn/render-and-commit#step-3-react-commits-changes-to-the-dom):

- Durante el **renderizado,** React llama a tus componentes para averiguar que debería estar en la pantalla.
- Durante la **confirmación,** React aplica los cambios a el DOM.

**En general, [no quieres](https://es.react.dev/learn/referencing-values-with-refs#best-practices-for-refs) acceder a las refs durante el renderizado**. Eso va también para las refs que tienen nodos DOM. **Durante el primer renderizado, los nodos DOM aún no han sido creados, entonces `ref.current` será `null`**. Y durante el renderizado de actualizaciones, los nodos DOM aún no se han actualizado. Es muy temprano para leerlos.

**React establece `ref.current` durante la confirmación**. Antes de actualizar el DOM, React establece los valores afectados de `ref.current` a `null`. Después de actualizar el DOM, React inmediatamente los establece en los nodos DOM correspondientes.

**Generalmente, vas a acceder a las refs desde los controladores de eventos.** Si quieres hacer algo con una ref, pero no hay un evento en particular para hacerlo, es posible que necesites un _Efecto_. Discutiremos los _Efectos_ en las próximas lecciones.

#### ⭐ Vaciando actualizaciones de estado sincrónicamente con flushSync

Considere un código como el siguiente, que agrega un nuevo _todo_ y desplaza la pantalla hasta el último hijo de la lista. **Observa cómo, por alguna razón, siempre se desplaza hacia el todo que estaba _justo antes_ del último que se ha agregado**.

```jsx
import { useState, useRef } from 'react';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    // Esto se ejecuta antes del renderizado 👇
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Agregar
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

Muestra:

![[2-manipular-el-dom-con-refs-5.png]]

Al ingresar algo en el input y pulsar en el botón, vemos cómo primero la pantalla se desliza y luego muestra la nueva tarea añadida:

![[2-manipular-el-dom-con-refs-6.png]]

El problema está con estas dos lineas:

```js
setTodos([ ...todos, newTodo]);
listRef.current.lastChild.scrollIntoView();
```

**En React, [las actualizaciones de estados se ponen en cola.](https://es.react.dev/learn/queueing-a-series-of-state-updates)** Generalmente, esto es lo que quieres. Sin embargo, **aquí causa un problema porque `setTodos` no actualiza el DOM inmediatamente. Entonces, en el momento en el que desplazas la lista al último elemento, el todo aún no ha sido agregado**. Esta es la razón por la que al desplazarse siempre se «retrasa» en un elemento.

**Para arreglar este problema, puedes forzar a React a actualizar («_flush_») el DOM sincrónicamente. Para hacer esto, importa `flushSync` del `react-dom` y _envuelve el actualizador de estado_ en una llamada a `flushSync`**:

```jsx
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

**Esto le indicará a React que actualice el DOM sincrónicamente justo después que el código envuelto en `flushSync` se ejecute**. Como resultado, el último todo ya estará en el DOM en el momento que intentes desplazarte hacia él.

```jsx
import { useState, useRef } from 'react';
import { flushSync } from 'react-dom';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    flushSync(() => { // 👈
      setText('');
      setTodos([ ...todos, newTodo]);      
    });
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Agregar
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

## ⭐ Mejores prácticas para la manipulación del DOM con refs 

**Las refs son una vía de escape. Sólo deberías usarlas cuando tengas que «_salirte de React_»**. **Ejemplos comunes de esto incluyen la gestión del foco, la posición del scroll, o una llamada a las API del navegador que React no expone**.

Si te limitas a acciones no destructivas como enfocar o desplazarte, no deberías encontrar ningún problema. Sin embargo, **si intentas modificar el DOM manualmente, puedes arriesgarte a entrar en conflicto con los cambios que React está haciendo**.

Para ilustrar este problema, este ejemplo incluye un mensaje de bienvenida y dos botones. El primer botón alterna su presencia usando [renderizado condicional](https://es.react.dev/learn/conditional-rendering) y [estado](https://es.react.dev/learn/state-a-components-memory), como normalmente lo harías en React. El segundo botón usa la [API del DOM `remove()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/remove) para eliminarlo forzadamente del DOM fuera del control de React.

Intenta presionar «Alternar con setState» unas cuantas veces. El mensaje debe desaparecer y aparecer otra vez. Luego presiona «Eliminar del DOM». Esto lo eliminará forzadamente. Finalmente, presiona «Alternar con setState»:

**App.js**
```jsx
import { useState, useRef } from 'react';

export default function Counter() {
  const [show, setShow] = useState(true);
  const ref = useRef(null);

  return (
    <div>
      <button
        onClick={() => {
          setShow(!show);
        }}>
        Alternar con setState
      </button>
      <button
        onClick={() => {
          ref.current.remove();
        }}>
        Eliminar del DOM
      </button>
      {show && <p ref={ref}>Hola Mundo</p>}
    </div>
  );
}
```

Muestra:

![[2-manipular-el-dom-con-refs-7.png]]

Al pulsar sobre el botón "Alternar con setState" el mensaje desaparece:

![[2-manipular-el-dom-con-refs-8.png]]

Al volverlo a pulsar el mensaje aparece de nuevo:

![[2-manipular-el-dom-con-refs-7.png]]

Si ahora pulsamos sobre el botón "Eliminar del DOM" el mensaje vuelve a desaparecer:

![[2-manipular-el-dom-con-refs-8.png]]

Pero si ahora volvemos a pulsar en el botón "Alternar con setState" nos dará un error:

![[2-manipular-el-dom-con-refs-9.png]]

Al parecido pero con un diferente mensaje de error ocurre si intentamos presionar el botón "Eliminar del DOM" cuando el mensaje no esté visible al haberlo ocultado pulsando el botón "Alternar con setState":

![[2-manipular-el-dom-con-refs-10.png]]

Después de que hayas eliminado el elemento DOM, intentar usar `setState` para mostrarlo de nuevo provocará un fallo. Esto se debe a que has cambiado el DOM, y React no sabe cómo seguir gestionándolo correctamente.

**Evita cambiar nodos DOM gestionados por React.** Modificar, agregar hijos, o eliminar hijos de elementos que son gestionados por React pueden traer resultados inconsistentes visuales o fallos como el de arriba.

Sin embargo, esto no quiere decir que no puedas en absoluto. Requiere de cuidado. **Puedes modificar de manera segura partes del DOM que React _no tenga motivos_ para actualizar.** Por ejemplo, si algún `<div>` siempre está vacío en el JSX, React no tendrá un motivo para tocar su lista de elementos hijos. Por lo tanto, es seguro agregar o eliminar manualmente elementos allí.

## Recapitulación

- Las refs son un concepto genérico, pero a menudo las vas a usar para almacenar elementos del DOM.
- Tú le indicas a React a poner un nodo DOM dentro de `myRef.current` pasándole `<div ref={myRef}>`.
- Normalmente, vas a usar las refs para acciones no destructivas como enfocar, desplazar, o medir elementos DOM.
- Un componente no expone sus nodos DOM por defecto. Puedes optar por exponer un nodo DOM usando `forwardRef` y pasando el segundo argumento `ref` a un nodo específico.
- Evita cambiar nodos DOM gestionados por React.
- Si modificas nodos DOM gestionados por React, modifica las partes en donde React no tenga motivos para actualizar.

