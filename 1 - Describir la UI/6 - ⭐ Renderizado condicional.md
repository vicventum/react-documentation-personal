> https://es.react.dev/learn/conditional-rendering

Tus componentes a menudo necesitarán mostrar diferentes cosas dependiendo de diferentes condiciones. En React, puedes renderizar JSX de forma condicional utilizando la sintaxis de JavaScript como las declaraciones `if`, `&&` y los operadores `? :`.

### Aprenderás

- Cómo devolver distinto JSX dependiendo de una condición
- Cómo incluir o excluir condicionalmente un fragmento de JSX
- Atajos de sintaxis condicional comunes que encontrarás en las bases de código de React

## Devolución condicional de JSX

Supongamos que tienes un componente `PackingList` que muestra varios `Items`, que pueden ser marcados como empaquetados o no:

**App.jsx**
```jsx
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista de equipaje de Sally Ride</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Traje de vuelo" 
        />
        <Item 
          isPacked={true} 
          name="Casco con dorado a la hoja" 
        />
        <Item 
          isPacked={false} 
          name="Fotografía de Tam" 
        />
      </ul>
    </section>
  );
}
```

Muestra:

![[6-renderizado-condicional-1.png]]

Observa que algunos de los componentes `Item` tienen su _prop_ `isPacked` asignada a `true` en lugar de `false`. Se desea añadir una marca de verificación (✔) a los elementos empaquetados si `isPacked={true}`.

Puedes escribir esto como una declaración [`if`/`else`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Statements/if...else) así:

```jsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

Si la prop `isPacked` es `true`, este código **devuelve un árbol JSX diferente**. Con este cambio, algunos de los elementos obtienen una marca de verificación al final:

**App.js**
```jsx
function Item({ name, isPacked }) {
  if (isPacked) { 👈
    return <li className="item">{name} ✔</li>; 👈
  }
  return <li className="item">{name}</li>; 👈
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista de equipaje de Sally Ride</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Traje de vuelo" 
        />
        <Item 
          isPacked={true} 
          name="Casco con dorado a la hoja" 
        />
        <Item 
          isPacked={false} 
          name="Fotografía de Tam" 
        />
      </ul>
    </section>
  );
}
```

Muestra:

![[6-renderizado-condicional-2.png]]

Prueba a editar lo que se devuelve en cualquiera de los dos casos y observa cómo cambia el resultado.

Observa cómo estás creando una lógica de ramificación con las sentencias `if` y `return` de JavaScript. En React, el flujo de control (como las condiciones) es manejado por JavaScript.

### Devolución de nada con `null` (no recomendado)

En algunas situaciones, no querrás mostrar nada en absoluto. Por ejemplo, digamos que no quieres mostrar elementos empaquetados en absoluto. Un componente debe devolver algo. En este caso, puedes devolver `null`:

```jsx
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

Si `isPacked` es verdadero, el componente no devolverá nada, `null`. En caso contrario, devolverá JSX para ser renderizado.

**App.js**
```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return null; // 👈
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista de equipaje de Sally Ride</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Traje de vuelo" 
        />
        <Item 
          isPacked={true} 
          name="Casco con dorado a la hoja" 
        />
        <Item 
          isPacked={false} 
          name="Fotografía de Tam" 
        />
      </ul>
    </section>
  );
}
```

Muestra:

![[6-renderizado-condicional-4.png]]

> [!important]
> En la práctica, devolver `null` en un componente no es común porque podría sorprender a un desarrollador que intente renderizarlo. Lo más frecuente es incluir o excluir condicionalmente el componente en el JSX del componente padre. Aquí se explica cómo hacerlo.

## Exclusión condicional de JSX

En el ejemplo anterior, controlabas qué árbol JSX (si es que había alguno) era devuelto por el componente. Es posible que ya hayas notado alguna duplicación en la salida de la renderización:

```jsx
<li className="item">{name} ✔</li>
```

es muy similar a

```jsx
<li className="item">{name}</li>
```

Ambas ramas condicionales devuelven `<li className="item">...</li>`:

```jsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

**Aunque esta duplicación no es perjudicial, podría hacer que tu código sea más difícil de mantener**. ¿Qué pasa si quieres cambiar el `className`? ¡Tendrías que hacerlo en dos lugares en tu código! En tal situación, podrías incluir condicionalmente un poco de JSX para hacer tu código más [DRY](https://es.wikipedia.org/wiki/No_te_repitas).

### Operador condicional (ternario) (`? :`

JavaScript tiene una sintaxis compacta para escribir una expresión condicional — el [operador condicional](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) u «_operador ternario_».

En lugar de esto:

```jsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

Puedes escribir esto:

```jsx
return (
  <li className="item">
    {isPacked ? name + ' ✔' : name}
  </li>
);
```

Puedes leerlo como _«si `isPacked` es verdadero, entonces (`?`) renderiza `name + ' ✔'`, de lo contrario (`:`) renderiza `name`»_)

> [!info]
> ### ¿Son estos dos ejemplos totalmente equivalentes?
> Si vienes de un entorno de programación orientada a objetos, podrías asumir que los dos ejemplos anteriores son sutilmente diferentes porque uno de ellos puede crear dos «instancias» diferentes de `<li>`. Pero los elementos JSX no son «instancias» porque no contienen ningún estado interno y no son nodos reales del DOM. Son descripciones ligeras, como los planos. Así que estos dos ejemplos, de hecho, _son_ completamente equivalentes. En [Preservar y reiniciar el estado](https://es.react.dev/learn/preserving-and-resetting-state) se explica en detalle cómo funciona esto.

Ahora digamos que quieres envolver el texto del elemento completado en otra etiqueta HTML, como `<del>` para tacharlo. Puedes añadir aún más líneas nuevas y paréntesis para que sea más fácil anidar más JSX en cada uno de los casos:

**App.js**
```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? ( {/*👈*/}
        <del>
          {name + ' ✔'}
        </del>
      ) : (
        name
      )}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista de equipaje de Sally Ride</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Traje de vuelo" 
        />
        <Item 
          isPacked={true} 
          name="Casco con dorado a la hoja" 
        />
        <Item 
          isPacked={false} 
          name="Fotografía de Tam" 
        />
      </ul>
    </section>
  );
}
```

Muestra:

![[6-renderizado-condicional-3.png]]

Este estilo funciona bien para condiciones simples, pero utilízalo con moderación. Si tus componentes se desordenan con demasiado marcado condicional anidado, considera la posibilidad de extraer componentes hijos para limpiar las cosas. En React, el marcado es una parte de tu código, por lo que puedes utilizar herramientas como variables y funciones para ordenar las expresiones complejas.

### Operador lógico AND (`&&`)

Otro atajo común que encontrarás es el [operador lógico AND (`&&`) de JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#:~:text=The%20logical%20AND%20(%20%26%20)%20operator,it%20returns%20a%20Boolean%20value.). Dentro de los componentes de React, a menudo surge cuando quieres renderizar algún JSX cuando la condición es verdadera, **o no renderizar nada en caso contrario.** Con `&&`, podrías renderizar condicionalmente la marca de verificación sólo si `isPacked` es `true`:

```jsx
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```

Puedes leer esto como _«si `isPacked`, entonces (`&&`) renderiza la marca de verificación, si no, no renderiza nada.»_

Aquí está en acción:

**App.js**
```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✔'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista de equipaje de Sally Ride</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Traje de vuelo" 
        />
        <Item 
          isPacked={true} 
          name="Casco con dorado a la hoja" 
        />
        <Item 
          isPacked={false} 
          name="Fotografía de Tam" 
        />
      </ul>
    </section>
  );
}
```

Muestra:

![[6-renderizado-condicional-2.png]]

Una [expresión JavaScript &&](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND) devuelve el valor de su lado derecho (en nuestro caso, la marca de verificación) si el lado izquierdo (nuestra condición) es `true`. Pero si la condición es `false`, toda la expresión se convierte en `false`. **React considera `false` como un «agujero» en el árbol JSX, al igual que `null` o `undefined`, y no renderiza nada en su lugar**.

> [!warning]
> ⭐ **No pongas números a la izquierda de `&&`.**
>
>Para comprobar la condición, JavaScript convierte el lado izquierdo en un booleano automáticamente. **Sin embargo, si el lado izquierdo es `0`, entonces toda la expresión obtiene ese valor (`0`), y React representará felizmente `0` en lugar de nada.**
>
>Por ejemplo, un error común es escribir código como `messageCount && <p>New messages</p>`. Es fácil suponer que no renderiza nada cuando `messageCount` es `0`, pero en realidad renderiza el propio `0`.
>
>Para arreglarlo, haz que el lado izquierdo sea un booleano: `messageCount > 0 && <p>New messages</p>`.

### Asignación condicional de JSX a una variable 

Cuando los atajos se interpongan en el camino de la escritura de código simple, prueba a utilizar una sentencia `if` y una variable. Puedes reasignar las variables definidas con [`let`](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Statements/let), así que empieza proporcionando el contenido por defecto que quieres mostrar, el nombre:

```jsx
let itemContent = name;
```

Utiliza una sentencia `if` para reasignar una expresión JSX a `itemContent` si `isPacked` es `true`:

```jsx
if (isPacked) {
  itemContent = name + " ✔";
}
```

[[4 - ⭐ JavaScript en JSX con llaves#⭐ Usando llaves Una ventana al mundo de JavaScript|Las llaves abren la «ventana a JavaScript».]] Inserta la variable con llaves en el árbol JSX devuelto, anidando la expresión previamente calculada dentro de JSX:

```jsx
<li className="item">
  {itemContent}
</li>
```

Este estilo es el más verboso, pero también el más flexible. Aquí está en acción:

**App.js**
```jsx
function Item({ name, isPacked }) {
  let itemContent = name; // 👈
  if (isPacked) { // 👈
    itemContent = name + " ✔";
  }
  return (
    <li className="item">
      {itemContent} // 👈
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista de equipaje de Sally Ride</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Traje de vuelo" 
        />
        <Item 
          isPacked={true} 
          name="Casco con dorado a la hoja" 
        />
        <Item 
          isPacked={false} 
          name="Fotografía de Tam" 
        />
      </ul>
    </section>
  );
}
```

Como antes, esto funciona no sólo para el texto, sino también para JSX arbitrario:

**App.js**
```jsx
function Item({ name, isPacked }) {
  let itemContent = name; // 👈
  if (isPacked) {
    itemContent = ( // 👈
      <del>
        {name + " ✔"}
      </del>
    );
  }
  return (
    <li className="item">
      {itemContent} // 👈
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista de equipaje de Sally Ride</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Traje de vuelo" 
        />
        <Item 
          isPacked={true} 
          name="Casco con dorado a la hoja" 
        />
        <Item 
          isPacked={false} 
          name="Fotografía de Tam" 
        />
      </ul>
    </section>
  );
}
```

Si no estás familiarizado con JavaScript, esta variedad de estilos puede parecer abrumadora al principio. Sin embargo, aprenderlos te ayudará a leer y escribir cualquier código JavaScript — ¡y no sólo los componentes de React! Escoge el que prefieras para empezar, y luego vuelve a consultar esta referencia si olvidas cómo funcionan los demás.

## Recapitulación

- En React, se controla la lógica de ramificación con JavaScript.
- Puedes devolver una expresión JSX condicionalmente con una sentencia `if`.
- Puedes guardar condicionalmente algún JSX en una variable y luego incluirlo dentro de otro JSX usando las llaves.
- En JSX, `{cond ? <A /> : <B />}` significa _«si `cond`, renderiza `<A />`, si no `<B />`»_.
- En JSX, `{cond && <A />}` significa _«si `cond`, renderiza `<A />`, si no, nada»_.
- Los atajos son comunes, pero no tienes que usarlos si prefieres el simple `if`.