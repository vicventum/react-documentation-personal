**Los componentes de React utilizan _props_ para comunicarse entre sí. Cada componente padre puede enviar información a sus componentes hijos mediante el uso de props. Las props pueden parecerte similares a los atributos HTML, pero permiten pasar cualquier valor de JavaScript a través de ellas, como objetos, arrays y funciones.

### Aprenderás

- Cómo pasar props a un componente
- Cómo acceder a las props desde un componente
- Cómo asignar valores predeterminados para las props
- Cómo pasar código JSX a un componente
- Cómo las props cambian con el tiempo

## Props conocidas 

Las props son los datos que se pasan a un elemento JSX. Por ejemplo, `className`, `src`, `alt`, `width` y `height` son algunas de las props que se pueden pasar a un elemento `<img>`:

**App.js**
```jsx
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function Profile() {
  return (
    <Avatar />
  );
}
```

Muestra:

![[5-pasar-props-a-un-componente-1.png]]

Las props que puedes utilizar con una etiqueta `<img>` están ya predefinidas (_ReactDOM_ se ajusta al [estándar HTML](https://www.w3.org/TR/html52/semantics-embedded-content.html#the-img-element)). **Sin embargo, puedes pasar cualquier prop a _tus propios_ componentes**, como `<Avatar>`, para personalizarlos. ¡Aquí te mostramos cómo hacerlo!

## Pasar props a un componente

En este código, el componente `Profile` no está pasando ninguna prop a su componente hijo, `Avatar`:

```jsx
export default function Profile() {
  return (
    <Avatar />
  );
}
```

Puedes pasar props al elemento `Avatar` en dos pasos.

### Paso 1: Pasar props al componente hijo

Primero, pasa algunas _props_ al elemento `Avatar`. Por ejemplo, vamos a asignar dos props: `person` (un objeto) y `size` (un número):

```jsx
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

> [!note]
> Si te resulta confuso el uso de llaves dobles después de `person=`, recuerda que [simplemente estamos pasando un objeto](https://es.react.dev/learn/javascript-in-jsx-with-curly-braces#using-double-curlies-css-and-other-objects-in-jsx) dentro de las llaves JSX.

Ahora puedes acceder a estas props dentro del componente `Avatar`.

### ⭐ Paso 2: Acceder a props dentro del componente hijo

Puedes acceder a estas props especificando sus nombres `person, size` separados por comas dentro de `({` y `})` justo después de `function Avatar`. Esto te permitirá utilizarlas dentro del código de `Avatar` como si fueran variables.

```jsx
function Avatar({ person, size }) {
  // puedes acceder a los valores de person y size desde aquí
}
```

Agrega lógica a `Avatar` que utilice las props `person` y `size` para la renderización, ¡y eso es todo!.

Ahora puedes configurar `Avatar` para que se renderice de diferentes maneras con distintas props. ¡Prueba ajustando los valores!

**App.js**
```jsx
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <div>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi', 
          imageId: 'YfeOqp2'
        }}
      />
      <Avatar
        size={80}
        person={{
          name: 'Aklilu Lemma', 
          imageId: 'OKS67lh'
        }}
      />
      <Avatar
        size={50}
        person={{ 
          name: 'Lin Lanying',
          imageId: '1bX5QH6'
        }}
      />
    </div>
  );
}
```

**utils.js**
```jsx
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

Las props te permiten considerar de forma independiente los componentes padre e hijo. Por ejemplo, **puedes modificar las props `person` o `size` dentro del componente `Profile` sin preocuparte por cómo serán utilizadas por el componente `Avatar`**. De manera similar, puedes cambiar la forma en que `Avatar` utiliza estas props sin necesidad de revisar el componente `Profile`.

Considera las props como «_controles_» que puedes ajustar. Cumplen el mismo papel que los argumentos de una función—de hecho, ¡**las props _son_ el único argumento de tu componente**! Las funciones de los componentes de React aceptan un único argumento, un objeto `props`:

```jsx
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

En general, no necesitas acceder al objeto completo de `props`, por lo que puedes desestructurarlo en props individuales.

>[!warning]
>**Asegurate de incluir el par de llaves `{` y `}`** dentro de `(` y `)` al declarar las props:
>
>```jsx
>function Avatar({ person, size }) {
>  // ...
>}
>```
>
>Esta sintaxis se conoce como [«desestructuración»](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#desempacar_campos_de_objetos_pasados_como_par%C3%A1metro_de_funci%C3%B3n) y es equivalente a acceder a propiedades de un parámetro de función:
>
>```jsx
>function Avatar(props) {
>  let person = props.person;
>  let size = props.size;
>   // ...
}
>```

## Asignar un valor predeterminado para una prop

Si quieres asignar un valor predeterminado para una prop en caso de que no se especifique ningún valor, **puedes hacerlo mediante la desestructuración colocando `=` seguido del valor predeterminado justo después del parámetro**:

```jsx
function Avatar({ person, size = 100 }) {
  // ...
}
```

Ahora, si renderizas `<Avatar person={...} />` sin la prop `size`, el valor de `size` se establecerá automáticamente en `100`.

> [!important]
>El valor predeterminado sólo se utilizará si falta la prop `size` o si se pasa `size={undefined}`. Sin embargo, si se pasa `size={null}` o `size={0}`, el valor predeterminado **no** se aplicará.

## Reenviar props con la sintaxis de propagación JSX

A veces, pasar props se vuelve muy repetitivo:

```jsx
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

No hay ningún problema en tener código repetitivo—ya que puede ser más legible. Sin embargo, en ocasiones, es posible que prefieras ser más conciso. Algunos componentes reenvían todas sus props a sus hijos, como lo hace `Profile` con `Avatar`. Dado que no utilizan directamente ninguna de sus props, tiene sentido utilizar una sintaxis de «propagación» más concisa:

```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

Esto permite reenviar todas las props de `Profile` a `Avatar` sin la necesidad de especificar cada una de ellas.

**Recuerda utilizar la sintaxis de propagación con moderación.** Si estás utilizando esta sintaxis en cada componente, es probable que algo no esté correctamente estructurado. En muchos casos, esto sugiere que deberías dividir tus componentes y pasar los hijos como elementos JSX separados. ¡Más información sobre esto a continuación!

> [!danger]
> Si bien esta sintaxis en válida, es recomendable no usarla, ya que es mejor escribir un código descriptivo aunque sea verboso, que uno más corto pero que pueda llevar a confusiones 
 
## ⭐ Pasar JSX como hijos: prop `children`

Es común anidar etiquetas nativas del navegador:

```html
<div>
  <img />
</div>
```

En ocasiones, querrás anidar tus propios componentes de la misma forma:

```jsx
<Card>
  <Avatar />
</Card>
```

**Al anidar contenido dentro de una etiqueta JSX, el componente padre recibirá ese contenido a través de una prop llamada `children`**. En el ejemplo a continuación, el componente `Card` recibe una prop `children` con el valor de `<Avatar />` y lo renderiza dentro de un div contenedor:

**App.js**
```jsx
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```

**Avatar.js**
```jsx
import { getImageUrl } from './utils.js';

export default function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

**utils.js**
```jsx
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

Prueba cambiando `<Avatar>` dentro de `<Card>` con algún texto para ver cómo el componente `Card` puede envolver cualquier contenido anidado. **No es necesario que el componente «sepa» qué se está renderizando dentro de él**. Este patrón flexible se puede observar en muchos casos.

**Puedes imaginar un componente con una prop `children` como si tuviera un «hueco» que puede ser «llenado» por sus componentes padres con JSX arbitrario**. La prop `children` suele utilizarse para crear envoltorios visuales como paneles, rejillas, etc.

![A puzzle-like Card tile with a slot for "children" pieces like text and Avatar](https://es.react.dev/images/docs/illustrations/i_children-prop.png)
>Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

## ⭐ Cómo las props cambian con el tiempo (y props inmutables)

El componente `Clock` que se muestra a continuación recibe dos props de su componente padre: `color` y `time`. (Se omite el código del componente padre porque utiliza [estado](https://es.react.dev/learn/state-a-components-memory), del cual no ahondaremos en este momento.)

Prueba cambiando el color en la lista desplegable que aparece a continuación:

**Clock.js**
```jsx
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}

```
Muestra

![[5-pasar-props-a-un-componente-2.png]]

Este ejemplo demuestra que **un componente puede recibir props que cambian a lo largo del tiempo.** ¡**Las props no siempre son estáticas**! Aquí, la prop `time` cambia a cada segundo, y la prop `color` cambia cuando se elige un color diferente. Las props reflejan los datos de un componente en cualquier momento, y no sólo al inicio.

**Sin embargo, las props son [inmutables](https://en.wikipedia.org/wiki/Immutable_object)**—un término de la informática que significa «_inalterable_». Si un componente necesita cambiar sus props (por ejemplo, en respuesta a una interacción del usuario o nuevos datos), debe «solicitar» a su componente padre que le pase _nuevas props_—¡un nuevo objeto! Las props antiguas se descartarán y eventualmente el motor de JavaScript liberará la memoria que ocupaban.

**No intentes «cambiar las props».** Cuando necesites responder al input del usuario (como cambiar el color seleccionado), deberás «establecer un estado», lo cual puedes aprender en [El estado: la memoria de un componente.](https://es.react.dev/learn/state-a-components-memory)

## Recapitulación

- Para pasar props, simplemente agrégalas al JSX, de la misma forma en que lo harías con los atributos HTML.
- Para acceder a las props, utiliza la sintaxis de desestructuración `function Avatar({ person, size })`.
- Puedes asignar un valor predeterminado como `size = 100`, que se utiliza para las props faltantes y `undefined`.
- Puedes reenviar todas las props con la sintaxis de propagación JSX `<Avatar {...props} />`, ¡pero no abuses de ella!
- Todo JSX anidado como `<Card><Avatar /></Card>` aparecerá como la prop `children` del componente `Card`.
- Las props son instantáneas de solo lectura en el tiempo: cada renderizado recibe una nueva versión de las props.
- No puedes cambiar las props. Si necesitas interactividad, tendrás que establecer un estado.Los componentes de React utilizan _props_ para comunicarse entre sí. Cada componente padre puede enviar información a sus componentes hijos mediante el uso de props. Las props pueden parecerte similares a los atributos HTML, pero permiten pasar cualquier valor de JavaScript a través de ellas, como objetos, arrays y funciones.

### Aprenderás

- Cómo pasar props a un componente
- Cómo acceder a las props desde un componente
- Cómo asignar valores predeterminados para las props
- Cómo pasar código JSX a un componente
- Cómo las props cambian con el tiempo
