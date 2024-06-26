>[!important]
> Usar _computed properties_ y lógica dentro de los controladores de eventos antes de usar un Efecto

Los Efectos son una vía de escape del paradigma de React. Te permiten «_salir_» de React y sincronizar tus componentes con algún sistema externo, como un _widget_ que no es de React, una red o el DOM del navegador. **Si no hay ningún sistema externo involucrado (por ejemplo, si deseas actualizar el estado de un componente cuando cambian ciertas _props_ o el estado), no deberías necesitar un Efecto. Eliminar Efectos innecesarios hará que tu código sea más fácil de seguir, se ejecute más rápido y sea menos propenso a errores**.

### Aprenderás

- Por qué y cómo eliminar Efectos innecesarios de tus componentes.
- Cómo almacenar en caché cálculos costosos sin utilizar Efectos.
- Cómo reiniciar y ajustar el estado del componente sin utilizar Efectos.
- Cómo compartir lógica entre controladores de eventos.
- Qué lógica debería ser trasladada a los controladores de eventos.
- Cómo notificar a los componentes padre acerca de cambios.

## Cómo eliminar Efectos innecesarios

Hay dos casos comunes en los cuales no necesitas utilizar Efectos:

- **No necesitas Efectos para transformar datos antes de renderizar.** Por ejemplo, **supongamos que deseas filtrar una lista antes de mostrarla. Podrías sentirte tentado/a a escribir un Efecto que actualice una variable de estado cuando cambie la lista. Sin embargo, esto es ineficiente**. Cuando actualizas el estado, React primero llama a las funciones de tu componente para calcular lo que debería mostrarse en la pantalla. Luego, React [«confirmará»](https://es.react.dev/learn/render-and-commit) estos cambios en el DOM, actualizando la pantalla. Después, React ejecuta tus Efectos. Si tu Efecto también actualiza inmediatamente el estado, ¡esto reinicia todo el proceso desde cero! **Para evitar pasadas de renderizado innecesarias, transforma todos los datos en el nivel superior de tus componentes. Ese código se volverá a ejecutar automáticamente cada vez que tus _props_ o estado cambien**.
- **No necesitas Efectos para manejar eventos del usuario.** Por ejemplo, supongamos que deseas enviar una solicitud POST `/api/buy` y mostrar una notificación cuando el usuario compra un producto. En el controlador de eventos del botón «Comprar», sabes exactamente lo que sucedió. Para el momento en que se ejecuta un Efecto, no sabes _qué_ hizo el usuario (por ejemplo, en qué botón se hizo clic). **Por esta razón, generalmente se manejan los eventos del usuario en los controladores de eventos correspondientes**.

Es _cierto_ que necesitas Efectos para [sincronizar](https://es.react.dev/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events) con sistemas externos. Por ejemplo, puedes escribir un Efecto que mantenga sincronizado un _widget_ de jQuery con el estado de React. También puedes obtener datos con Efectos, por ejemplo, puedes sincronizar los resultados de búsqueda con la consulta de búsqueda actual. Ten en cuenta que los [_frameworks_](https://es.react.dev/learn/start-a-new-react-project#production-grade-react-frameworks) modernos proporcionan mecanismos más eficientes y nativos para obtener datos que escribir Efectos directamente en tus componentes.

Para ayudarte a desarrollar la intuición adecuada, ¡veamos algunos ejemplos concretos comunes!

### ⭐ Actualización del estado basada en _props_ o estado - Computed Properties

Supongamos que t**ienes un componente con dos variables de estado: `firstName` y `lastName`. Deseas calcular un `fullName` a partir de ellos concatenándolos**. Además, te gustaría que `fullName` se actualice cada vez que `firstName` o `lastName` cambien. Tu primer instinto podría ser agregar una variable de estado `fullName` y actualizarla en un Efecto:

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Evitar: estado redundante y Efecto innecesario 👈
  const [fullName, setFullName] = useState(''); // 👈
  useEffect(() => { // 👈
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

Esto es más complicado de lo necesario. También es ineficiente: realiza un pase de renderización completo con un valor obsoleto para `fullName`, y luego se vuelve a renderizar inmediatamente con el valor actualizado. Elimina la variable de estado y el Efecto, y ==en su lugar puedes simular el comportamiento de una _computed property_==:

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Correcto: calculado durante el renderizado.
  const fullName = firstName + ' ' + lastName; // 👈
  // ...
}
```

==**Cuando algo puede calcularse a partir de las _props_ o el estado existente, [no lo pongas en el estado](https://es.react.dev/learn/choosing-the-state-structure#avoid-redundant-state). En su lugar, calcúlalo durante el renderizado (con un _computed property_).**== Esto hace que tu código sea más rápido (evitas las actualizaciones adicionales «en cascada»), más simple (eliminas código innecesario) y menos propenso a errores (evitas errores causados por diferentes variables de estado desincronizadas entre sí). Si este enfoque te resulta nuevo, [Pensar en React](https://es.react.dev/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state) explica qué debe ir en el estado.

### ⭐ `useMemo` - Almacenamiento en caché de cálculos costosos - Computed Properties

> [!important] 
> Con el lanzamiento del _Compilador de React_ en la versión 19 de React, el uso de hooks como `useMemo` y `useCallback` para la optimización de cálculos, quedarán deprecados, ya que dichas optimizaciones serán realizadas por el compilador

Este componente calcula `visibleTodos` tomando los `todos` que recibe a través de _props_ y filtrándolos según la _prop_ `filter`. Podrías sentirte tentado/a de almacenar el resultado en el estado y actualizarlo desde un Efecto:

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 Evitar: estado redundante y Efecto innecesario 
  const [visibleTodos, setVisibleTodos] = useState([]); // 👈
  useEffect(() => { // 👈
    setVisibleTodos(getFilteredTodos(todos, filter)); // 👈
  }, [todos, filter]); // 👈

  // ...
}
```

Al igual que en el ejemplo anterior, esto es innecesario e ineficiente. Primero, elimina el estado y el Efecto:

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ Esto está bien si getFilteredTodos() no es lento.
  const visibleTodos = getFilteredTodos(todos, filter); // 👈
  // ...
}
```

**Usualmente, ¡este código está bien! Pero tal vez `getFilteredTodos()` sea lento o tengas muchos `todos`**. En ese caso, no querrás recalcular `getFilteredTodos()` si alguna variable de estado no relacionada, como `newTodo`, ha cambiado.

==Puedes almacenar en caché (o [«memoizar»](https://es.wikipedia.org/wiki/Memoizaci%C3%B3n)) un cálculo costoso envolviéndolo en un Hook de React [`useMemo`](https://es.react.dev/reference/react/useMemo)==:

```jsx
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => { // 👈
    // ✅ No se vuelve a ejecutar a menos que cambien todos o filter.
    return getFilteredTodos(todos, filter); // 👈
  }, [todos, filter]); // 👈
  // ...
}
```

O, escrito en una sola línea:

```jsx
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ No se vuelve a ejecutar getFilteredTodos() a menos que cambien todos o filter.
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]); // 👈
  // ...
}
```

**Esto le indica a React que no deseas que la función interna se vuelva a ejecutar a menos que `todos` o `filter` hayan cambiado.** React recordará el valor de devolución de `getFilteredTodos()` durante el renderizado inicial. Durante los siguientes renderizados, verificará si `todos` o `filter` son diferentes. Si son iguales que la última vez, `useMemo` devolverá el último resultado almacenado. Pero si son diferentes, React llamará nuevamente a la función interna (y almacenará su resultado).

La función que envuelves en [`useMemo`](https://es.react.dev/reference/react/useMemo) se ejecuta durante el renderizado, por lo que esto solo funciona para [cálculos puros.](https://es.react.dev/learn/keeping-components-pure)

#### ¿Cómo determinar si un cálculo es costoso? 

En general, a menos que estés creando o iterando sobre miles de objetos, probablemente no es costoso. Si deseas tener más confianza, puedes agregar un registro en la consola para medir el tiempo que se tarda en ejecutar una pieza de código:

```jsx
console.time('filter array'); // 👈
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array'); // 👈
```

R**ealiza la interacción que estás midiendo (por ejemplo, escribir en el campo de texto (_input_)). Luego, verás registros en la consola como `filter array: 0.15ms`. Si el tiempo total registrado suma una cantidad significativa (digamos, `1ms` o más), podría tener sentido memoizar ese cálculo.** Como experimento, puedes envolver el cálculo en `useMemo` para verificar si el tiempo total registrado ha disminuido para esa interacción o no:

```jsx
console.time('filter array');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // Se omite si todos y filter no han cambiado
}, [todos, filter]);
console.timeEnd('filter array');
```

**`useMemo` no hará que el _primer_ renderizado sea más rápido. Solo te ayuda a evitar trabajo innecesario en las actualizaciones posteriores**.

Ten en cuenta que tu máquina probablemente es más rápida que la de tus usuarios, por lo que es una buena idea probar el rendimiento con una ralentización artificial. Por ejemplo, Chrome ofrece una opción de [Limitación de CPU](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) para esto.

También ten en cuenta que medir el rendimiento en desarrollo no te dará los resultados más precisos. (Por ejemplo, cuando [Modo Estricto](https://es.react.dev/reference/react/StrictMode) está activado, verás que cada componente se renderiza dos veces en lugar de una). Para obtener los tiempos más precisos, construye tu aplicación para producción y pruébala en un dispositivo similar al que usan tus usuarios.

### ⭐ Reiniciar todo el estado cuando una _prop_ cambia - Usando `keys` para indicar componentes distintos

Este componente `ProfilePage` recibe una _prop_ `userId`. La página contiene un _input_ (entrada) de comentario, y tú usas una variable de estado `comment` para mantener este valor. Un día, tú te das cuenta de un problema: cuando navegas de un perfil a otro, el estado `comment` no se reinicia. Como resultado, es fácil publicar accidentalmente un comentario en el perfil de un usuario equivocado. Para arreglar el problema, tú quieres borrar la variable de estado `comment` cada vez que el `userId` cambie:

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Evitar: Reiniciar el estado en un cambio de prop dentro de un Efecto.
  useEffect(() => { // 👈
    setComment(''); // 👈
  }, [userId]); // 👈
  // ...
}
```

**Esto es ineficiente porque `ProfilePage` y sus hijos se renderizarán primero con el valor obsoleto, y luego se volverán a renderizar**. También es complicado porque tendrías que hacer esto en _cada_ componente que tenga algún estado dentro de `ProfilePage`. Por ejemplo, si la UI de comentarios está anidada, también querrías quitar el estado de los comentarios anidados.

**==En su lugar, puedes indicarle a React que el perfil de cada usuario es conceptualmente un perfil _diferente_ al proporcionarle una _key_ explícita==**. Divide tu componente en dos y pasa un atributo _`key`_ desde el componente externo al interno:

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId} // 👈
    />
  );
}

function Profile({ userId }) { // 👈
  // ✅ Esto y cualquier otro estado a continuación se reiniciarán automáticamente cuando cambie la key.
  const [comment, setComment] = useState(''); // 👈
  // ...
}
```

>[!important]
>**Normalmente, React preserva el estado cuando el mismo componente se renderiza en el mismo lugar. Al pasar `userId` como una _`key`_ al componente `Profile`, le estás indicando a React que trate dos componentes `Profile` con diferentes `userId` como dos componentes diferentes que no deben compartir ningún estado.** Cada vez que cambie la _key_ (que has establecido como `userId`), React recreará el DOM y [reiniciará el estado](https://es.react.dev/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key) del componente `Profile` y de todos sus hijos. Ahora, el campo `comment` se borrará automáticamente al navegar entre perfiles.

Ten en cuenta que en este ejemplo, solo el componente `ProfilePage` externo es exportado y visible para otros archivos en el proyecto. Los componentes que renderizan `ProfilePage` no necesitan pasar la _key_; simplemente pasan `userId` como una _prop_ regular. El hecho de que `ProfilePage` lo pase como una _`key`_ al componente interno `Profile` es un detalle de implementación.

### ⭐ Ajustar algún estado cuando cambia una _prop_ - Computed Properties

A veces, es posible que desees reiniciar o ajustar una parte del estado cuando cambie una _prop_, pero no todo el estado.

Este componente `List` recibe una lista de `items` como prop y mantiene el _item_ seleccionado en la variable de estado `selection`. Deseas reiniciar la `selection` a `null` cada vez que la _prop_ `items` reciba un _array_ diferente:

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Evitar: Ajustar el estado en un cambio de prop dentro de un Efecto.
  useEffect(() => { // 👈
    setSelection(null); // 👈
  }, [items]); // 👈
  // ...
}
```

**Esto, también, no es ideal. Cada vez que cambian los `items`, el componente `List` y sus componentes hijos se renderizarán inicialmente con un valor obsoleto de `selection`**. Luego, React actualizará el DOM y ejecutará los Efectos. Finalmente, la llamada a `setSelection(null)` provocará otra nueva renderización del componente `List` y sus componentes hijos, reiniciando todo este proceso nuevamente.

Comienza por eliminar el Efecto. **En su lugar, ajusta el estado directamente durante el renderizado**:

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Mejor: Ajusta el estado durante el renderizado.
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

[Almacenar información de renderizados previos](https://es.react.dev/reference/react/useState#storing-information-from-previous-renders) como se muestra en este ejemplo puede ser difícil de entender, pero es mejor que actualizar el mismo estado en un Efecto. En el ejemplo anterior, `setSelection` se llama directamente durante un renderizado. React volverá a renderizar el componente `List` _inmediatamente_ después de salir del bloque de `return`. React aún no ha renderizado los hijos de `List` ni ha actualizado el DOM, lo que permite a los hijos de `List` omitir el renderizado del valor obsoleto de `selection`.

Cuando actualizas un componente durante el renderizado, React descarta el JSX devuelto y vuelve a intentar el renderizado de inmediato. Para evitar reintentos en cascada muy lentos, React solo te permite actualizar el estado del _mismo_ componente durante el renderizado. Si intentas actualizar el estado de otro componente durante el renderizado, verás un error. Una condición como `items !== prevItems` es necesaria para evitar bucles. Puedes ajustar el estado de esta manera, pero otros efectos secundarios (como cambios en el DOM o establecer tiempos de espera) debe mantenerse en los controladores de eventos o en Efectos para [mantener los componentes puros.](https://es.react.dev/learn/keeping-components-pure)

**Aunque este patrón es más eficiente que un Efecto, la mayoría de los componentes tampoco lo necesitan.** Sin importar cómo lo hagas, ajustar el estado basado en _props_ u otro estado hace que el flujo de datos sea más difícil de entender y depurar. **==Siempre verifica si puedes [reiniciar todo el estado con una _key_](https://es.react.dev/learn/you-might-not-need-an-effect#resetting-all-state-when-a-prop-changes) o [calcular todo durante el renderizado](https://es.react.dev/learn/you-might-not-need-an-effect#updating-state-based-on-props-or-state) en su lugar==**. Por ejemplo, en lugar de almacenar (y reiniciar) el _ítem_ seleccionado, puedes almacenar el _ítem ID_ seleccionado:

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Mejor: Calcular todo durante el renderizado.
  const selection = items.find(item => item.id === selectedId) ?? null; // 👈
  // ...
}
```

**Ahora no hay necesidad de «ajustar» el estado en lo absoluto. Si el _item_ con el ID seleccionado está en la lista, permanecerá seleccionado. Si no lo está, la `selection` calculada durante el renderizado será `null` porque no se encontró ningún _item_ coincidente**. Este comportamiento es diferente, pero se podría decir que es mejor porque la mayoría de los cambios en `items` preservan la selección.

### ⭐ Compartir lógica entre controladores de eventos - Insertar lógica directamente en los controladores de eventos, no usar Efectos para ello

Supongamos que tienes una página de producto con dos botones (Comprar y Pagar) que permiten comprar ese producto. Deseas mostrar una notificación cada vez que el usuario agrega el producto al carrito. Llamar a `showNotification()` en los controladores de clic de ambos botones se siente repetitivo, por lo que podrías sentir la tentación de colocar esta lógica en un Efecto:

```jsx
function ProductPage({ product, addToCart }) {
  // 🔴 Evitar: Lógica específica del evento dentro de un Efecto.
  useEffect(() => { // 👈
    if (product.isInCart) { // 👈
      showNotification(`Added ${product.name} to the shopping cart!`); // 👈
    }
  }, [product]); // 👈

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

**Este Efecto es innecesario. También es muy probable que cause errores**. Por ejemplo, supongamos que tu aplicación «recuerda» el carrito de compras entre las recargas de página. Si agregas un producto al carrito una vez y actualizas la página, la notificación aparecerá de nuevo. Seguirá apareciendo cada vez que actualices la página del producto. Esto se debe a que `product.isInCart` ya será `true` en la carga de la página, por lo que el Efecto anterior llamará a `showNotification()`.

**Cuando no estés seguro si algún código debe estar en un Efecto o en un controlador de eventos, pregúntate _por qué_ este código necesita ejecutarse. ==Usa Efectos solo para el código que debe ejecutarse _porque_ el componente fue mostrado al usuario, no por una acción directa de este, para ello mejor usar los controladores de eventos directamente==.** En este ejemplo, la notificación debería aparecer porque el usuario _presionó el botón_, ¡no porque la página fue mostrada! Elimina el Efecto y coloca la lógica compartida en una función llamada desde ambos controladores de eventos:

```jsx
function ProductPage({ product, addToCart }) {
  // ✅ Correcto: La lógica específica del evento se llama desde los controladores de eventos.
  function buyProduct() { // 👈
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct(); // 👈
  }

  function handleCheckoutClick() {
    buyProduct(); // 👈
    navigateTo('/checkout');
  }
  // ...
}
```

Esto no solo elimina el Efecto innecesario, sino que también corrige el error.

### ⭐ Enviar una solicitud POST  - No usar Efectos para código que pueda ejecutarse en controladores de eventos

Este componente `Form` envía dos tipos de solicitudes POST. Envía un evento de analítica cuando se monta. Cuando completas el formulario y haces clic en el botón «Enviar», enviará una solicitud POST al punto final `/api/register`:

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Correcto: Esta lógica debe ejecutarse porque el componente fue mostrado al usuario.
  useEffect(() => { // 👈
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 Evitar: Lógica específica de evento dentro de un Efecto
  const [jsonToSubmit, setJsonToSubmit] = useState(null); // 👈
  useEffect(() => { // 👈
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

**Aplicaremos el mismo criterio que en el ejemplo anterior**.

**La solicitud POST de analítica debe permanecer en un Efecto. Esto se debe a que la _razón_ para enviar el evento de analítica es que el formulario se mostró**. (Puede dispararse dos veces en desarrollo, pero [ver aquí](https://es.react.dev/learn/synchronizing-with-effects#sending-analytics) para aprender cómo manejarlo).

**Sin embargo, la solicitud POST a `/api/register` no es causada por el formulario siendo _mostrado_. Solo deseas enviar la solicitud en un momento específico: cuando el usuario presiona el botón**. Debería suceder solo durante _esa interacción particular_. Elimina el segundo Efecto y coloca esa solicitud POST dentro del controlador de eventos:

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Correcto: Esta lógica se ejecuta porque el componente fue mostrado al usuario.
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Correcto: La lógica específica del evento se encuentra en el controlador de eventos.
    post('/api/register', { firstName, lastName }); // 👈
  }
  // ...
}
```

>[!important]
>Cuando decidas si colocar cierta lógica en un controlador de eventos o en un Efecto, la pregunta principal que debes responder es _qué tipo de lógica_ es desde la perspectiva del usuario. Si esta lógica es causada por una interacción particular, mantenla en el controlador de eventos. Si es causada por el usuario _visualizando_ el componente en la pantalla, mantenla en el Efecto.

### Cadenas de cálculos - Usar computed properties y controladores de eventos

A veces podrías sentirte tentado a encadenar Efectos que ajustan cada uno una parte del estado basándose en otro estado:

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Evitar: Cadenas de Efectos que ajustan el estado solo para activarse entre sí.
  useEffect(() => { // 👈
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => { // 👈
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => { // 👈
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => { // 👈
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

Hay dos problemas con este código.

**Un problema es que es muy ineficiente:** el componente (y su hijo) deben volver a renderizarse entre cada llamada a `set` en la cadena. En el ejemplo anterior, en el peor caso (`setCard` → renderizado → `setGoldCardCount` → renderizado → `setRound` → renderizado → `setIsGameOver` → renderizado), hay tres renderizados innecesarios del árbol hacia abajo.

Incluso si no fuera lento, a medida que evoluciona tu código, te encontrarás con casos en los que la «cadena» que escribiste no se ajusta a los nuevos requisitos. Imagina que estás agregando una forma de recorrer el historial de los movimientos del juego. Lo harías actualizando cada variable de estado a un valor del pasado. Sin embargo, establecer el estado de `card` a un valor del pasado volvería a activar la cadena de Efectos y cambiaría los datos que estás mostrando. Este tipo de código suele ser rígido y frágil.

**En este caso, es mejor calcular lo que puedas durante el proceso de renderizado y ajustar el estado en el controlador de eventos**:

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calcula lo que puedas durante el proceso de renderizado.
  const isGameOver = round > 5; // 👈

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calcula todo el próximo estado en el controlador de eventos.
    setCard(nextCard);
    if (nextCard.gold) { // 👈
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

**Esto es mucho más eficiente**. Además, si implementas una forma de ver el historial del juego, ahora podrás establecer cada variable de estado en un movimiento del pasado sin activar la cadena de Efectos que ajusta cada otro valor. Si necesitas reutilizar la lógica entre varios controladores de eventos, puedes [extraer una función](https://es.react.dev/learn/you-might-not-need-an-effect#sharing-logic-between-event-handlers) y llamarla desde esos controladores.

Recuerda que dentro de los controladores de eventos, [el estado se comporta como una instantánea.](https://es.react.dev/learn/state-as-a-snapshot) Por ejemplo, incluso después de llamar a `setRound(round + 1)`, la variable `round` reflejará el valor en el momento en que el usuario hizo clic en el botón. Si necesitas usar el siguiente valor para cálculos, defínelo manualmente como `const nextRound = round + 1`.

>[!warning]
>**En algunos casos, _no puedes_ calcular el siguiente estado directamente en el controlador de eventos**. Por ejemplo, imagina un formulario con múltiples menús desplegables donde las opciones del siguiente menú desplegable dependen del valor seleccionado en el menú desplegable anterior. **En este caso, una cadena de Efectos es apropiada porque estás sincronizando con la red**.

### ⭐ Inicializar la aplicación

Alguna lógica solo debería ejecutarse una vez cuando se carga la aplicación.

**Podrías sentirte tentado a colocarla en un Efecto en el componente de nivel superior**:

```jsx
function App() {
  // 🔴 Evitar: Efectos con lógica que solo deben ejecutarse una vez.
  useEffect(() => { // 👈
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []); // 👈
  // ...
}
```

**Sin embargo, rápidamente descubrirás que esto [se ejecuta dos veces en desarrollo](https://es.react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development). Esto puede causar problemas**, por ejemplo, tal vez invalide el _token_ de autenticación porque la función no fue diseñada para ser llamada dos veces. **En general, tus componentes deberían ser resistentes a ser montados de nuevo. Esto incluye tu componente de nivel superior `App`**.

Aunque en la práctica en producción es posible que nunca se vuelva a montar, seguir las mismas restricciones en todos los componentes facilita mover y reutilizar el código. ==**Si alguna lógica debe ejecutarse _una vez por carga de la aplicación_ en lugar de _una vez por montaje del componente_, agrega una variable de nivel superior para llevar un registro de si ya se ha ejecutado:**==

```jsx
let didInit = false; // 👈

function App() {
  useEffect(() => {
    if (!didInit) { // 👈
      didInit = true;
      // ✅ Se ejecuta solo una vez por carga de la aplicación.
      loadDataFromLocalStorage();
      checkAuthToken();
    } // 👈
  }, []);
  // ...
}
```

**También puedes ejecutarlo durante la inicialización del módulo y antes de que la aplicación se renderice**:

```jsx
if (typeof window !== 'undefined') { // Comprueba si estamos ejecutándolo en el navegador. 👈
   // ✅ Solo se ejecuta una vez por carga de la aplicación
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

>[!important]
> **El código en el nivel superior se ejecuta una vez cuando se importa tu componente, incluso si no se llega a renderizar. Para evitar ralentización o comportamientos inesperados al importar componentes arbitrarios, no abuses de este patrón**. Mantén la lógica de inicialización a nivel de la aplicación en módulos de componentes _root_, como `App.js`, o en el punto de entrada de tu aplicación.

### ⭐ Notificar a los componentes padre sobre cambios de estado - Usar controladores de eventos

Digamos que estás escribiendo un componente `Toggle` con un estado interno `isOn` que puede ser `true` o `false`. Hay algunas formas diferentes de alternarlo (haciendo clic o arrastrando). **Quieres notificar al componente padre cada vez que el estado interno del `Toggle` cambie, por lo que expones un evento `onChange` y lo llamas desde un Efecto**:

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Evitar: El controlador `onChange` se ejecuta demasiado tarde.
  useEffect(() => { // 👈
    onChange(isOn); // 👈
  }, [isOn, onChange]) // 👈

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

**Como mencionamos anteriormente, esto no es ideal**. El `Toggle` actualiza su estado primero, y React actualiza la pantalla. Luego, React ejecuta el Efecto, que llama a la función `onChange` pasada desde un componente padre. Ahora el componente padre actualizará su propio estado, iniciando otro proceso de renderizado. Sería mejor hacer todo en un solo paso.

**Elimina el Efecto y, en su lugar, actualiza el estado de _ambos_ componentes dentro del mismo controlador de eventos**:

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ Correcto: Realiza todas las actualizaciones durante el evento que las causó
    setIsOn(nextIsOn); // 👈
    onChange(nextIsOn); // 👈
  }

  function handleClick() {
    updateToggle(!isOn); // 👈
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true); // 👈
    } else {
      updateToggle(false); // 👈
    }
  }

  // ...
}
```

**Con este enfoque, tanto el componente `Toggle` como su componente padre actualizan su estado durante el evento**. React [agrupa las actualizaciones](https://es.react.dev/learn/queueing-a-series-of-state-updates) de diferentes componentes juntas, por lo que solo habrá un pase de renderizado.

**También podrías eliminar completamente el estado y, en su lugar, recibir `isOn` desde el componente padre**:

```jsx
// ✅ También correcto: el componente está completamente controlado por su padre
function Toggle({ isOn, onChange }) { // 👈
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

[«Levantar el estado"](https://es.react.dev/learn/sharing-state-between-components)" permite que el componente padre controle completamente el `Toggle` al alternar el estado del propio componente padre. Esto significa que el componente padre deberá contener más lógica, pero en general habrá menos estado con el que preocuparse. **==Siempre que intentes mantener sincronizadas dos variables de estado diferentes, ¡intenta levantar el estado en su lugar!==**

### ⭐ Pasar datos al componente padre - Mejor que el componente padre obtenga los datos y se los pase al hijo

Este componente `Child` obtiene algunos datos y luego los pasa al componente `Parent` en un Efecto:

```jsx
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Evitar: Pasar datos al padre en un Efecto
  useEffect(() => { // 👈
    if (data) {
      onFetched(data); // 👈
    }
  }, [onFetched, data]); // 👈
  // ...
}
```

En React, los datos fluyen desde los componentes padres hacia sus hijos. Cuando ves algo incorrecto en la pantalla, puedes rastrear de dónde proviene la información siguiendo la cadena de componentes hacia arriba hasta encontrar qué componente pasa la _prop_ incorrecta o tiene el estado incorrecto. **Cuando los componentes hijos actualizan el estado de sus componentes padres en Efectos, el flujo de datos se vuelve muy difícil de rastrear. Dado que tanto el hijo como el padre necesitan los mismos datos, permite que el componente padre obtenga esos datos y los _pase hacia abajo_ al hijo en su lugar**:

```jsx
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Correcto: Pasando datos hacia abajo al hijo.
  return <Child data={data} />; // 👈
}

function Child({ data }) {
  // ...
}
```

Esto es más simple y mantiene el flujo de datos predecible: los datos fluyen hacia abajo desde el padre hacia el hijo.

### Suscripción a un almacén externo [](https://es.react.dev/learn/you-might-not-need-an-effect#subscribing-to-an-external-store "Link for Suscripción a un almacén externo")

A veces, tus componentes pueden necesitar suscribirse a algunos datos fuera del estado de React. Estos datos podrían provenir de una biblioteca de terceros o de una API incorporada en el navegador. Dado que estos datos pueden cambiar sin que React lo sepa, es necesario suscribir manualmente tus componentes a ellos. Esto se hace frecuentemente con un Efecto, por ejemplo:

```
function useOnlineStatus() {  // No es lo ideal: Suscripción manual a un almacén en un Efecto.  const [isOnline, setIsOnline] = useState(true);  useEffect(() => {    function updateState() {      setIsOnline(navigator.onLine);    }    updateState();    window.addEventListener('online', updateState);    window.addEventListener('offline', updateState);    return () => {      window.removeEventListener('online', updateState);      window.removeEventListener('offline', updateState);    };  }, []);  return isOnline;}function ChatIndicator() {  const isOnline = useOnlineStatus();  // ...}
```

Aquí, el componente se suscribe a un almacén de datos externos (en este caso, la API `navigator.onLine` del navegador). Dado que esta API no existe en el servidor (por lo que no se puede utilizar para el HTML inicial), inicialmente el estado se establece en `true`. Cada vez que el valor de ese almacén de datos cambia en el navegador, el componente actualiza su estado.

Aunque es común utilizar Efectos para esto, React tiene un Hook específicamente diseñado para suscribirse a un almacén de datos externos que se prefiere en su lugar. Elimina el Efecto y reemplázalo con una llamada a [`useSyncExternalStore`](https://es.react.dev/reference/react/useSyncExternalStore):

```
function subscribe(callback) {  window.addEventListener('online', callback);  window.addEventListener('offline', callback);  return () => {    window.removeEventListener('online', callback);    window.removeEventListener('offline', callback);  };}function useOnlineStatus() {  // ✅ Bien: Suscribirse a un almacén externo con un Hook incorporado.  return useSyncExternalStore(    subscribe, // React no volverá a suscribirse mientras pases la misma función.    () => navigator.onLine, // Cómo obtener el valor en el cliente.    () => true // Cómo obtener el valor en el servidor.  );}function ChatIndicator() {  const isOnline = useOnlineStatus();  // ...}
```

Este enfoque es menos propenso a errores que la sincronización manual de datos mutables al estado de React con un Efecto. Típicamente, escribirás un Hook personalizado como `useOnlineStatus()` como se muestra arriba, para que no necesites repetir este código en los componentes individuales. [Lee más sobre cómo suscribirte a almacenes externos desde componentes React.](https://es.react.dev/reference/react/useSyncExternalStore)

### Obtención de datos [](https://es.react.dev/learn/you-might-not-need-an-effect#fetching-data "Link for Obtención de datos")

Muchas aplicaciones utilizan Efectos para iniciar la obtención de datos. Es bastante común escribir un Efecto para obtener datos de esta manera:

```
function SearchResults({ query }) {  const [results, setResults] = useState([]);  const [page, setPage] = useState(1);  useEffect(() => {    // 🔴 Evitar: Obtener datos sin lógica de limpieza.    fetchResults(query, page).then(json => {      setResults(json);    });  }, [query, page]);  function handleNextPageClick() {    setPage(page + 1);  }  // ...}
```

No _necesitas_ mover esta solicitud (_fetch_) a un controlador de eventos.

Esto puede parecer una contradicción con los ejemplos anteriores donde necesitabas poner la lógica en los controladores de eventos. Sin embargo, considera que no es _el evento de escritura_ la razón principal para realizar la solicitud (_fetch_). Los campos de búsqueda a menudo se precargan desde la URL, y el usuario podría navegar hacia atrás y adelante sin tocar el campo de búsqueda.

No importa de dónde provengan `page` y `query`. Mientras este componente sea visible, deseas mantener `results` [sincronizado](https://es.react.dev/learn/synchronizing-with-effects) con los datos de la red para la `page` y `query` actuales. Por eso es un Efecto.

Sin embargo, el código anterior tiene un error. Imagina que escribes «hola» rápidamente. Entonces la `query` cambiará de «h», a «ho», «hol», y «hola». Esto iniciará búsquedas separadas, pero no hay garantía sobre el orden en que llegarán las respuestas. Por ejemplo, la respuesta «hol» puede llegar _después_ de la respuesta «hola». Como «hol» llamará a `setResults()` al final, estarás mostrando los resultados de búsqueda incorrectos. Esto se llama una [«condición de carrera»](https://es.wikipedia.org/wiki/Condici%C3%B3n_de_carrera): dos solicitudes diferentes «compitieron» entre sí y llegaron en un orden diferente al que esperabas.

**Para solucionar la condición de carrera, necesitas [agregar una función de limpieza](https://es.react.dev/learn/synchronizing-with-effects#fetching-data) para ignorar respuestas obsoletas:**

```
function SearchResults({ query }) {  const [results, setResults] = useState([]);  const [page, setPage] = useState(1);  useEffect(() => {    let ignore = false;    fetchResults(query, page).then(json => {      if (!ignore) {        setResults(json);      }    });    return () => {      ignore = true;    };  }, [query, page]);  function handleNextPageClick() {    setPage(page + 1);  }  // ...}
```

Esto asegura que cuando tu Efecto obtiene datos, todas las respuestas excepto la última solicitada serán ignoradas.

Manejar las condiciones de carrera no es la única dificultad al implementar la obtención de datos. También podrías considerar el almacenamiento en caché de las respuestas (para que el usuario pueda hacer clic en «Atrás» y ver la pantalla anterior instantáneamente), cómo obtener datos en el servidor (para que el HTML renderizado inicialmente por el servidor contenga el contenido obtenido en lugar de un indicador de carga (_spinner_)), y cómo evitar cascadas de red (para que un hijo pueda obtener datos sin tener que esperar por cada padre).

**Estos problemas aplican a cualquier biblioteca de UI, no solo a React. Resolverlos no es trivial, por eso los [frameworks](https://es.react.dev/learn/start-a-new-react-project#production-grade-react-frameworks) modernos ofrecen mecanismos incorporados más eficientes de obtención de datos que obtener datos en Efectos.**

Si no utilizas un framework (y no quieres construir el tuyo propio) pero te gustaría hacer que la obtención de datos desde Efectos sea más cómoda, considera extraer tu lógica de obtención de datos en un Hook personalizado, como en este ejemplo:

```
function SearchResults({ query }) {  const [page, setPage] = useState(1);  const params = new URLSearchParams({ query, page });  const results = useData(`/api/search?${params}`);  function handleNextPageClick() {    setPage(page + 1);  }  // ...}function useData(url) {  const [data, setData] = useState(null);  useEffect(() => {    let ignore = false;    fetch(url)      .then(response => response.json())      .then(json => {        if (!ignore) {          setData(json);        }      });    return () => {      ignore = true;    };  }, [url]);  return data;}
```

Probablemente también querrás agregar lógica para el manejo de errores y para rastrear si el contenido está cargando. Puedes construir un Hook como este por ti mismo o utilizar una de las muchas soluciones ya disponibles en el ecosistema de React. **Aunque por sí solo esto no será tan eficiente como usar el mecanismo incorporado de obtención de datos de un framework, al mover la lógica de obtención de datos a un Hook personalizado, será más fácil adoptar una estrategia eficiente de obtención de datos más adelante.**

En general, cada vez que te veas obligado a escribir Efectos, mantén un ojo para identificar cuándo puedes extraer una funcionalidad en un Hook personalizado con una API más declarativa y específica, como `useData` mencionado anteriormente. Cuantas menos llamadas directas a `useEffect` tengas en tus componentes, más fácil te resultará mantener tu aplicación.

## Recapitulación[](https://es.react.dev/learn/you-might-not-need-an-effect#recap "Link for Recapitulación")

- Si puedes calcular algo durante el renderizado, no necesitas un Efecto.
- Para almacenar en caché cálculos costosos, utiliza `useMemo` en lugar de `useEffect`.
- Para reiniciar el estado de todo el árbol de componentes, pasa una _`key`_ diferente a este.
- Para reiniciar una porción del estado en respuesta a un cambio de _prop_, establécelo durante el renderizado.
- El código que se ejecuta porque un componente fue _mostrado_ debería estar en Efectos, el resto debería estar en eventos.
- Si necesitas actualizar el estado de varios componentes, es mejor hacerlo durante un solo evento.
- Siempre que intentes sincronizar variables de estado en diferentes componentes, considera levantar el estado.
- Puedes obtener datos con Efectos, pero necesitas implementar limpieza para evitar condiciones de carrera