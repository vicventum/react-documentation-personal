Por lo general, pasarás información desde un componente padre a un componente hijo por medio de props. Sin embargo, **pasar props puede convertirse en una tarea verbosa e inconveniente si tienes que pasarlas a través de múltiples componentes**, o si varios componentes en tu aplicación necesitan la misma información. **El _contexto_ permite que cierta información del componente padre esté disponible en cualquier componente del árbol que esté por debajo de él sin importar qué tan profundo sea y sin pasar la información explícitamente por medio de props**.

### Aprenderás

- Qué es «perforación de props»
- Cómo reemplazar el paso repetitivo de props con contexto
- Casos de uso comunes para el contexto
- Alternativas comunes al contexto

## El problema con pasar props

[Pasar props](https://es.react.dev/learn/passing-props-to-a-component) es una gran manera de enviar explícitamente datos a través del árbol de la UI a componentes que los usen.

No obstante, **pasar props puede convertirse en una tarea verbosa e inconveniente cuando necesitas enviar algunas props profundamente a través del árbol, o si múltiples componentes necesitan de las mismas**. El ancestro común más cercano podría estar muy alejado de los componentes que necesitan los datos, y [elevar el estado](https://es.react.dev/learn/sharing-state-between-components) tan alto puede ocasionar la situación llamada «_perforación de props_».

![[6-context-pasar-datos-en-profundidad-1.png|670]]

¿No sería grandioso si existiese alguna **forma de «_teletransportar_» datos a componentes en el árbol que lo necesiten sin tener que pasar props? ¡Con el contexto de React es posible!**

## ⭐ _Context_: una alternativa a pasar props

**El contexto permite que el componente padre provea datos al árbol entero debajo de él**. Hay muchas utilidades para el contexto. Este es un solo ejemplo. Considera el componente `Heading` que acepta `level` como su tamaño:

**App.js**
```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Título</Heading>
      <Heading level={2}>Encabezado</Heading>
      <Heading level={3}>Sub-encabezado</Heading>
      <Heading level={4}>Sub-sub-encabezado</Heading>
      <Heading level={5}>Sub-sub-sub-encabezado</Heading>
      <Heading level={6}>Sub-sub-sub-sub-encabezado</Heading>
    </Section>
  );
}
```

**Section.js**
```jsx
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Heading.js**
```jsx
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

Muestra:

![[6-context-pasar-datos-en-profundidad-2.png]]

Supongamos que quieres múltiples encabezados (_headings_) dentro del mismo componente `Section` para siempre tener el mismo tamaño:

**App.js**
```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Título</Heading>
      <Section>
        <Heading level={2}>Encabezado</Heading>
        <Heading level={2}>Encabezado</Heading>
        <Heading level={2}>Encabezado</Heading>
        <Section>
          <Heading level={3}>Sub-encabezado</Heading>
          <Heading level={3}>Sub-encabezado</Heading>
          <Heading level={3}>Sub-encabezado</Heading>
          <Section>
            <Heading level={4}>Sub-sub-encabezado</Heading>
            <Heading level={4}>Sub-sub-encabezado</Heading>
            <Heading level={4}>Sub-sub-encabezado</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```


**Section.js**
```jsx
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Heading.js**
```jsx
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

Muestra:

![[6-context-pasar-datos-en-profundidad-3.png]]


Actualmente, estás pasando la prop `level` a cada `<Heading>` separadamente:

```jsx
<Section>
  <Heading level={3}>Acerca de</Heading>
  <Heading level={3}>Fotos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

**Sería genial si pudieras pasar la prop `level` al componente `<Section>` y removerlo del `<Heading>`**. De esta forma podrías reforzar que todos los encabezados tengan el mismo tamaño en una misma sección (_section_):

```jsx
<Section level={3}>
  <Heading>Acerca de</Heading>
  <Heading>Fotos</Heading>
  <Heading>Videos</Heading>
</Section>
```

¿Pero como podría el componente `<Heading>` conocer el `level` de su `<Section>` más cercano? **Eso requeriría alguna forma en la que el hijo «pediría» datos desde algún lugar arriba en el árbol.**

No podrías lograrlo únicamente con props. Aquí es donde el contexto entra a jugar. Lo conseguirás en tres pasos:

1. **Crear** un contexto (puedes llamarlo `LevelContext`, ya que es para el `level` de los encabezados)
2. **Usar** ese contexto desde el componente que necesita los datos (`Heading` usará `LevelContext`)
3. **Proveer** ese contexto desde el componente que especifica los datos (`Section` proveerá `LevelContext`)

El contexto permite que en un padre (incluso uno distante) provea algunos datos a la totalidad del árbol dentro de él.

![[Pasted image 20240623213054.png|670]]
### Paso 1: Crear el contexto 

Primeramente, necesitas **crear el contexto. Necesitarás exportarlo desde un archivo para que tus componentes lo puedan usar**:

**LevelContext.js**
```jsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
```
  

El único parámetro que se le pasa a `createContext` es el valor _predeterminado_. En este caso, `1` se refiere al nivel de encabezado más grande, pero puedes pasar cualquier valor (incluso un objeto). Ya verás la importancia del valor predeterminado en el siguiente paso.

### Paso 2: Usar el contexto 

Importa el Hook `useContext` desde React y tu contexto:

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';
```

Actualmente, el componente `Heading` lee `level` con props:

```jsx
export default function Heading({ level, children }) {
  // ...
}
```

En su lugar, remueve la prop `level` y lee el valor desde el contexto que acabas de importar, `LevelContext`, usando el hook `useContext` de React:

```jsx
export default function Heading({ children }) {
  const level = useContext(LevelContext); // 👈
  // ...
}
```

`useContext` es un Hook. Así como `useState` y `useReducer`, únicamente puedes llamar a un Hook inmediatamente adentro de un componente de React (no dentro de ciclos o condiciones). **`useContext` le dice a React que el componente `Heading` quiere leer el contexto `LevelContext`.**

Ahora que el componente `Heading` no tiene una prop `level`, ya no tienes que pasarla a `Heading` en tu JSX de esta forma:

```jsx
<Section>
  <Heading level={4}>Sub-sub-encabezado</Heading>
  <Heading level={4}>Sub-sub-encabezado</Heading>
  <Heading level={4}>Sub-sub-encabezado</Heading>
</Section>
```

Actualiza el JSX para que sea `Section` el que recibe la prop:

```jsx
<Section level={4}>
  <Heading>Sub-sub-encabezado</Heading>
  <Heading>Sub-sub-encabezado</Heading>
  <Heading>Sub-sub-encabezado</Heading>
</Section>
```

Como recordatorio, esta es la estructura que estabas intentando que funcionara:

**App.js**
```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}> // 👈
      <Heading>Título</Heading>
      <Section level={2}> // 👈
        <Heading>Encabezado</Heading>
        <Heading>Encabezado</Heading>
        <Heading>Encabezado</Heading>
        <Section level={3}> // 👈
          <Heading>Sub-encabezado</Heading>
          <Heading>Sub-encabezado</Heading>
          <Heading>Sub-encabezado</Heading>
          <Section level={4}> // 👈
            <Heading>Sub-sub-encabezado</Heading>
            <Heading>Sub-sub-encabezado</Heading>
            <Heading>Sub-sub-encabezado</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```


**Section.js**
```jsx
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Heading.js**
```jsx
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

Muestra:

![[6-context-pasar-datos-en-profundidad-3.png]]

**Nota que este ejemplo no funciona, ¡Aún!** Todos los encabezados tienen el mismo tamaño porque **pese a que estás _usando_ el contexto, no lo has _proveído_ aún.** ¡React no sabe dónde obtenerlo!

**Si no provees el contexto, React usará el valor predeterminado que especificaste en el paso previo. En este ejemplo, especificaste `1` como el parámetro de `createContext`, entonces `useContext(LevelContext)` devuelve `1`, ajustando todos los encabezados a `<h1>`**. Arreglemos este problema haciendo que cada `Section` provea su propio contexto.

### Paso 3: Proveer el contexto 

El componente `Section` actualmente renderiza sus hijos:

**Section.jsx**
```jsx
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Envuélvelos con un proveedor de contexto para proveer `LevelContext` a ellos**:

**Section.jsx**
```jsx
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}> // 👈
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

Esto le dice a React: «_si cualquier componente adentro de este `<Section>` pregunta por `LevelContext`, envíales este `level`_». **El componente usará el valor del `<LevelContext.Provider>` más cercano en el árbol de la UI encima de él**.

**App.js**
```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}> // 👈
      <Heading>Título</Heading>
      <Section level={2}> // 👈
        <Heading>Encabezado</Heading>
        <Heading>Encabezado</Heading>
        <Heading>Encabezado</Heading>
        <Section level={3}> // 👈
          <Heading>Sub-encabezado</Heading>
          <Heading>Sub-encabezado</Heading>
          <Heading>Sub-encabezado</Heading>
          <Section level={4}> // 👈
            <Heading>Sub-sub-encabezado</Heading>
            <Heading>Sub-sub-encabezado</Heading>
            <Heading>Sub-sub-encabezado</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

**LevelContext.js**
```jsx
import { createContext } from 'react'; // 👈

export const LevelContext = createContext(1); // 👈
```

**Section.js**
```jsx
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}> // 👈
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

**Heading.js**
```jsx
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

Muestra:

![[6-context-pasar-datos-en-profundidad-3.png]]

Es el mismo resultado del código original, ¡pero no tuviste que pasar la prop `level` a cada componente `Heading`! En su lugar, el componente «comprende» su nivel de encabezado al preguntarle al `Section` más cercano de arriba:

1. Pasas la prop `level` al `<Section>`.
2. `Section` envuelve a sus hijos con `<LevelContext.Provider value={level}>`.
3. `Heading` pregunta el valor más cercano de arriba de `LevelContext` por medio de `useContext(LevelContext)`.

## ⭐ Usar y proveer el contexto desde el mismo componente

Actualmente, aún puedes especificar el `level` de cada sección manualmente:

```jsx
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

**Debido a que el contexto te permite leer información desde un componente de arriba, cada `Section` podría leer el `level` del `Section` de arriba, y pasar `level + 1` hacia abajo automáticamente**. Así es como lo podrías conseguir:

**Section.jsx**
```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext); // 👈
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}> // 👈
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

**Con este cambio, no es necesario pasar la prop `level` al `<Section>` o al `<Heading>`**:

**App.js**
```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section> // 👈
      <Heading>Título</Heading>
      <Section> // 👈
        <Heading>Encabezado</Heading>
        <Heading>Encabezado</Heading>
        <Heading>Encabezado</Heading>
        <Section> // 👈
          <Heading>Sub-encabezado</Heading>
          <Heading>Sub-encabezado</Heading>
          <Heading>Sub-encabezado</Heading>
          <Section> // 👈
            <Heading>Sub-sub-encabezado</Heading>
            <Heading>Sub-sub-encabezado</Heading>
            <Heading>Sub-sub-encabezado</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

**LevelContext.js**
```jsx
import { createContext } from 'react'; // 👈

export const LevelContext = createContext(1); // 👈
```

**Section.js**
```jsx
import { useContext } from 'react'; // 👈
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext); // 👈
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}> // 👈
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

**Heading.js**
```jsx
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

Muestra:

![[6-context-pasar-datos-en-profundidad-3.png]]

**Ahora, tanto el `Heading` como el `Section` leen el `LevelContext` para averiguar qué tan «_profundos_» están**. El `Section` envuelve sus hijos con el `LevelContext` para especificar que cualquier componente adentro de él está a un nivel más «_profundo_».

>[!tip]
> Este ejemplo usa niveles de encabezados porque muestran visualmente cómo componentes anidados pueden sobrescribir contextos. Sin embargo, los contextos son útiles para otros casos de uso también. Puedes pasar hacia abajo cualquier información necesitada por el subárbol entero: el color actual del tema, el usuario actual que inició sesión, entre otros.

## El contexto pasa a través de componentes intermedios

**Puedes insertar tantos componentes como desees entre el componente que provee el contexto y el componente que lo usa**. Esto incluye tanto componentes integrados como `<div>` como componentes construidos por ti.

En este ejemplo, el mismo componente `Post` (con un borde discontinuo) es renderizado en dos distintos niveles anidados. Nota que el `<Heading>` que está adentro tiene el nivel automáticamente desde el `<Section>` más cercano:

**App.js**
```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function ProfilePage() {
  return (
    <Section>
      <Heading>Mi perfil</Heading>
      <Post
        title="¡Hola viajero!"
        body="Lee sobre mis aventuras."
      />
      <AllPosts />
    </Section>
  );
}

function AllPosts() {
  return (
    <Section>
      <Heading>Publicaciones</Heading>
      <RecentPosts />
    </Section>
  );
}

function RecentPosts() {
  return (
    <Section>
      <Heading>Publicaciones recientes</Heading>
      <Post
        title="Sabores de Lisboa"
        body="¡...esos pastéis de nata!"
      />
      <Post
        title="Buenos Aires a ritmo de tango"
        body="¡Me encantó!"
      />
    </Section>
  );
}

function Post({ title, body }) {
  return (
    <Section isFancy={true}>
      <Heading>
        {title}
      </Heading>
      <p><i>{body}</i></p>
    </Section>
  );
}
```

**Section.js**
```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children, isFancy }) {
  const level = useContext(LevelContext);
  return (
    <section className={
      'section ' +
      (isFancy ? 'fancy' : '')
    }>
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

**Heading.js**
```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading must be inside a Section!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

**LevelContext.js**
```jsx
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

![[6-context-pasar-datos-en-profundidad-4.png]]

No necesitaste hacer nada especial para esta tarea. Cada `Section` especifica el contexto para el árbol adentro de él, por lo que puedes insertar un `<Heading>` en cualquier lado, y tendrá el tamaño correcto.

**El contexto te permite crear componentes que se «adaptan a sus alrededores» y se despliegan de forma diferente dependiendo de _dónde_ (o en otras palabras, _en cuál contexto_) están siendo renderizados.**

El funcionamiento de los contextos te podría recordar a la [herencia de CSS.](https://developer.mozilla.org/es/docs/Web/CSS/inheritance) En CSS, puedes especificar `color: blue` para un `<div>`, y cualquier nodo DOM adentro de él, no importa qué tan profundo esté, heredará ese color a no ser de que otro nodo DOM en el medio lo sobrescriba con `color: green`. Asimismo, en React **la única forma de sobrescribir un contexto que viene desde arriba es envolviendo sus hijos con un proveedor de contexto que tenga un valor distinto**.

En CSS, diversas propiedades como `color` y `background-color` no se sobrescriben entre ellas. Puedes definir la propiedad `color` de todos los `<div>` a `red` sin impactar `background-color`. Similarmente, **diversos contextos de React no se sobrescriben entre ellos mismos.** **Cada contexto que creas con `createContext()` está completamente separado de los otros, y une los componentes usando y proveyendo _ese_ contexto en particular**. Un componente podría usar o proveer muchos contextos diferentes sin ningún problema.

## ⭐ Antes de usar contexto 

¡El uso contexto resulta muy atractivo! Sin embargo, esto también significa que fácilmente puedes terminar abusando de él. **Solo porque necesitas pasar algunas props a varios niveles en profundidad no significa que debas poner esa información en un contexto.**

Aquí hay algunas alternativas que podrías considerar antes de usar el contexto:

1. **Empieza [pasando props.](https://es.react.dev/learn/passing-props-to-a-component)** Si tus componentes no son triviales, no es inusual pasar muchas props hacia abajo a través de muchos componentes. Podría considerarse tedioso, ¡pero deja bien claro cuáles componentes usan cuáles datos! La persona dándole mantenimiento a tu código estará agradecida de que hiciste el flujo de datos explícito con props.
2. **Extraer componentes y [pasarles el JSX como `children`](https://es.react.dev/learn/passing-props-to-a-component#passing-jsx-as-children).** Si pasas algunos datos a través de muchas capas de componentes intermedios que no usan esos datos (y lo único que hacen es pasarlos hacia abajo), esto muchas veces significa que olvidaste extraer algunos componentes sobre la marcha. Por ejemplo, quizá pasaste algunas props como `posts` a componentes visuales que no las usan directamente, como lo puede ser `<Layout posts={posts} />`. En su lugar, haz que `Layout` tome `children` como prop, y renderiza `<Layout><Posts posts={posts} /></Layout>`. Esto reduce la cantidad de capas que hay entre el componente que especifica los datos y el componente que los necesita.

Si ninguna de estas alternativas funcionan bien para ti, considera el contexto.

## Casos de uso para el contexto

- **Temas:** Si tus aplicaciones permiten que los usuarios cambien la apariencia (por ejemplo, modo oscuro), puedes poner un proveedor de contexto en el primer nivel de tu aplicación, y usar ese contexto en componentes que necesiten ajustar su comportamiento visual.
- **Cuenta actual:** Muchos componentes podrían necesitar saber el usuario actual que inició sesión. Ponerlo en un contexto lo hace conveniente para leerlo desde cualquier lado del árbol. Algunas aplicaciones también te permiten manejar múltiples cuentas al mismo tiempo (por ejemplo, dejar un comentario con un usuario distinto). En esos casos, puede ser conveniente envolver parte de la UI con un proveedor anidado que tenga una cuenta actual diferente.
- **Enrutamiento:** La mayoría de las soluciones de enrutamiento usan contexto internamente para mantener la ruta actual. Así es como cada enlace «sabe» si está activo o no. Si construyes tu propio enrutador, podrías necesitar hacerlo también.
- **Gestionar estados:** A medida que tu aplicación crece, podrías terminar con muchos estados cerca de la parte superior de tu aplicación. Muchos componentes distantes de abajo podrían querer cambiarlos. Es común [usar un reducer con un contexto](https://es.react.dev/learn/scaling-up-with-reducer-and-context) para gestionar estados complejos y pasarlos a componentes lejanos sin mucha molestia.

El contexto no está limitado a valores estáticos. Si pasas un valor distinto en el siguiente render, ¡React actualizará todos los componentes debajo que lean el contexto! Es por esto que muchas veces el contexto es usado en combinación con estados.

En general, si alguna información es necesitada por componentes lejanos en diferentes partes del árbol, es un buen indicador de que el contexto te ayudará.

## ⭐ Recapitulación

- El contexto permite que el componente provea alguna información al árbol completo debajo de él.
- Para pasar un contexto:
    1. Crear y exportar el contexto con `export const MyContext = createContext(defaultValue)`.
    2. Pasarlo al Hook `useContext(MyContext)` para leerlo en cualquier componente hijo, sin importar qué tan profundo es.
    3. Envolver los hijos con `<MyContext.Provider value={...}>` para proveerlo desde el padre.
- El contexto pasa a través de cualquier componente en el medio.
- El contexto te permite escribir componentes que se «adaptan a sus alrededores».
- Antes de usar contexto, trata de pasar props o pasar JSX como `children`.