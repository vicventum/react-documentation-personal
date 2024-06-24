Para que tus componentes se muestren en pantalla, antes deben ser renderizados por React. Entender los pasos de este proceso te ayudará a pensar en cómo se ejecuta tu código y a explicar su comportamiento.

### Aprenderás

- Qué significa el renderizado en React
- Cuándo y por qué React renderiza un componente
- Las etapas de la visualización de un componente en la pantalla
- Por qué el renderizado no siempre produce una actualización del DOM

Imagina que tus componentes son cocineros en la cocina, montando sabrosos platos a partir de los ingredientes. En este escenario, React es el camarero que hace las peticiones de los clientes y les trae sus pedidos. Este proceso de solicitud y servicio de UI tiene tres pasos:

1. **Desencadenamiento** de un renderizado (entrega del pedido del cliente a la cocina)
2. **Renderizado** del componente (preparación del pedido en la cocina)
3. **Confirmación** con el DOM (poner el pedido sobre la mesa)
    
![[3-renderizado-y-confirmacion-1.png]]
>Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

## Paso 1: Desencadenar un renderizado

Hay dos razones por las que un componente debe ser renderizado:

1. Es el **renderizado inicial** del componente.
2. El estado del componente (o de uno de sus ancestros) **ha sido actualizado.**

### Renderizado inicial

Cuando tu aplicación se inicia, necesitas activar el renderizado inicial. Frameworks y sandboxes a veces ocultan este código, pero s**e hace con una llamada a [`createRoot`](https://es.react.dev/reference/react-dom/client/createRoot) con el nodo DOM de destino, y luego con otra llamada a su método `render` con tu componente**:

**index.js**
```jsx
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Image />); // 👈
```

**Image.js**
```jsx
export default function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' de Eduardo Catalano: una gigantesca escultura floral metálica con pétalos reflectantes"
    />
  );
}
```

Muestra:

![[3-renderizado-y-confirmacion-3.png]]

### Rerenderizados cuando se actualiza el estado

Una vez que el componente ha sido renderizado inicialmente, **puede desencadenar más renderizados actualizando su estado con la [función `set`**](https://es.react.dev/reference/react/useState#setstate) del `useState`. Al actualizar el estado de tu componente, se pone en cola automáticamente un renderizado. (Puedes imaginarte esto como un cliente de un restaurante que pide té, postre y todo tipo de cosas después de poner su primer pedido, dependiendo del estado de su sed o hambre).

![[3-renderizado-y-confirmacion-2.png]]
>Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

## Paso 2: React renderiza tus componentes

Después de activar un renderizado, React llama a tus componentes para averiguar qué mostrar en la pantalla. **Un «renderizado» consiste en que React haga una llamada a tus componentes.**

- **En el renderizado inicial,** React llamará al componente raíz.
- **Para los siguientes renderizados,** React llamará al componente de función cuya actualización de estado desencadenó el renderizado.

Este proceso es recursivo: **si el componente actualizado devuelve algún otro componente, React renderizará _ese_ componente a continuación, y si ese componente también devuelve algo, renderizará _ese_ componente a continuación, y así sucesivamente**. El proceso continuará hasta que no haya más componentes anidados y React sepa exactamente qué debe mostrarse en pantalla.

En el siguiente ejemplo, React llamará a `Gallery()` y a `Image()` varias veces:

**index.js**
```jsx
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Gallery />);
```

**Gallery.js**
```jsx
export default function Gallery() {
  return (
    <section>
      <h1>Esculturas inspiradoras</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' de Eduardo Catalano: una gigantesca escultura floral metálica con pétalos reflectantes"
    />
  );
}
```


Muestra:

![[3-renderizado-y-confirmacion-4.png]]

- **Durante el renderizado inicial,** React [creará los nodos del DOM](https://developer.mozilla.org/docs/Web/API/Document/createElement) para `<section>`, `<h1>`, y tres etiquetas `<img>`.
- **Durante un rerenderizado,** React calculará cuáles de sus propiedades, si es que hay alguna, han cambiado desde el renderizado anterior. No hará nada con esa información hasta el siguiente paso, la fase de confirmación.

> [!warning]
>El renderizado debe ser siempre un [cálculo puro](https://es.react.dev/learn/keeping-components-pure):
>
>- **Misma entrada, misma salida.** Dadas las mismas entradas, un componente debería devolver siempre el mismo JSX. (Cuando alguien pide una ensalada con tomates, no debería recibir una ensalada con cebollas).
>- **Se ocupa de sus propios asuntos.** No debería cambiar ningún objeto o variable que existiera antes del renderizado. (Una orden no debe cambiar la orden de nadie más).
>
>De lo contrario, puedes encontrarte con errores confusos y un comportamiento impredecible a medida que tu base de código crece en complejidad. Cuando se desarrolla en «Modo estricto», React llama dos veces a la función de cada componente, lo que puede ayudar a aflorar los errores causados por funciones impuras.

> [!info]
> #### Optimización del rendimiento
>
>El comportamiento por defecto de renderizar todos los componentes anidados dentro del componente actualizado no es óptimo para el rendimiento si el componente actualizado está muy alto en el árbol. Si se encuentra con un problema de rendimiento, hay varias formas de resolverlo descritas en la sección de [Rendimiento](https://es.reactjs.org/docs/optimizing-performance.html). **¡No optimices antes de tiempo!**

## Paso 3: React confirma los cambios en el DOM

Después de renderizar (llamar) tus componentes, React modificará el DOM.

- **Para el renderizado inicial,** React utilizará la API del DOM [`appendChild()`](https://developer.mozilla.org/docs/Web/API/Node/appendChild) para poner en pantalla todos los nodos del DOM que ha creado.
- **Para los rerenderizados,** React aplicará las operaciones mínimas necesarias (¡calculadas durante el renderizado!) para hacer que el DOM coincida con la última salida del renderizado.

**React sólo cambia los nodos del DOM si hay una diferencia entre los renderizados.** Por ejemplo, este es un componente que se vuelve a renderizar con diferentes props pasadas desde su padre cada segundo. Fíjate en que puedes añadir algún texto en el `<input>`, actualizando su `valor`, pero el texto no desaparece cuando el componente se vuelve a renderizar:

**Clock.js**
```jsx
export default function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>
      <input />
    </>
  );
}
```

Muestra:

![[3-renderizado-y-confirmacion-5.png]]

**Esto funciona porque durante este último paso, React sólo actualiza el contenido de `<h1>` con el nuevo `time`. Ve que el `<input>` aparece en el JSX en el mismo lugar que la última vez, así que React no toca el `<input>`-¡ni su `valor`!**

## Epílogo: La pintura del navegador

Después de que el renderizado haya terminado y React haya actualizado el DOM, el navegador volverá a pintar la pantalla. Aunque este proceso se conoce como «renderizado del navegador», nos referiremos a él como «pintado» para evitar confusiones en el resto de esta documentación.

![Un navegador pinta una 'naturaleza muerta con elemento de tarjeta'.](https://es.react.dev/images/docs/illustrations/i_browser-paint.png)
> Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

## Recapitulación

- Cualquier actualización de pantalla en una aplicación React ocurre en tres pasos:
    1. Desencadenamiento
    2. Renderizado
    3. Confirmación
- Puedes utilizar el modo estricto para encontrar errores en tus componentes
- React no toca el DOM si el resultado del renderizado es el mismo que la última vez