Hay ocasiones en las que quieres que el **estado de dos componentes cambien siempre juntos. Para hacerlo, elimina el estado de los dos, muévelo al padre común más cercano y luego pásalo a través de props**. Esto se conoce como _elevar el estado (lifting state up)_, y es una de las cosas más comunes que harás al escribir código React.

### Aprenderás

- Cómo compartir el estado entre componentes por elevación
- Qué son los componentes controlados y no controlados

## Elevar el estado con un ejemplo

En este ejemplo, el componente padre `Accordion` renderiza dos componentes `Panel`:

- `Accordion`
    - `Panel`
    - `Panel`

Cada componente `Panel` tiene un estado booleano ‘`isActive`’ que determina si su contenido es visible.

Presiona el botón _Mostrar_ en ambos paneles:

**App.js**
```jsx
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Mostrar
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Alma Ata, Kazajistán</h2>
      <Panel title="Acerca de">
        Con una población de unos 2 millones de habitantes, Alma Ata es la mayor ciudad de Kazajistán. De 1929 a 1997 fue su capital.
      </Panel>
      <Panel title="Etimología">
        El nombre proviene de <span lang="kk-KZ">алма</span>, palabra en kazajo que significa "manzana" y suele traducirse como "lleno de manzanas". De hecho, se cree que la región que rodea a Alma Ata es el hogar ancestral de la manzana, y se considera que este fruto silvestre <i lang="la">Malus sieversii</i> es un candidato probable para el ancestro de la manzana doméstica moderna.
      </Panel>
    </>
  );
}
```

Muestra: 

![[3-compartir-estado-entre-componentes-1.png]]

Al pulsar el botón del primer recuadro:

![[3-compartir-estado-entre-componentes-2.png]]

Al pulsar el botón del segundo recuadro:

![[3-compartir-estado-entre-componentes-3.png]]

Observa que pulsar el botón de un panel no afecta al otro: son independientes.

![[3-compartir-estado-entre-componentes-4.png|650]]


**Pero ahora digamos que quieres cambiarlo para que solo se mantenga expandido un panel a la vez.** Con ese diseño, al expandir el segundo panel se debería colapsar el primero. ¿Cómo lo harías?

Para coordinar estos dos paneles, es necesario «_elevar su estado_» a un componente padre en tres pasos:

1. **Remueve** el estado de los componentes hijos.
2. **Transfiere** los datos codificados desde el padre común.
3. **Añade** estado al padre común y pasarlo hacia abajo junto con los controladores de eventos.

Esto permitirá que el componente `Accordion` coordine ambos `Panel` y sólo expanda uno a la vez.

### Paso 1: Elimina el estado de los componentes hijos

**Le darás el control de `isActive` del `Panel` a su componente padre**. Esto significa que **el componente padre pasará `isActive` al `Panel` como prop**. Empieza por **eliminar esta línea** del componente `Panel`:

```jsx
const [isActive, setIsActive] = useState(false);
```

Y en su lugar, añade `isActive` a la lista de props del `Panel`:

```jsx
function Panel({ title, children, isActive }) {
```

Ahora el componente padre de `Panel` puede _controlar_ `isActive` [pasándolo como prop.](https://es.react.dev/learn/passing-props-to-a-component) A la inversa, el componente `Panel` ahora no tiene _ningún control_ sobre el valor de `isActive`—¡ahora depende del componente padre!

### Paso 2: Pasa los datos codificados desde el componente padre común

Para elevar el estado, debes **localizar el componente padre común más cercano de _ambos_ componentes hijos que deseas coordinar**:

- `Accordion` _(padre común más cercano)_
    - `Panel`
    - `Panel`

**En este ejemplo, es el componente `Accordion`, dado que está por encima de ambos paneles y puede controlar sus props**, se convertirá en la «_fuente de la verdad_» para saber qué panel está actualmente activo. Haz que el componente `Accordion` pase un valor codificado de `isActive` (por ejemplo, `true`) a ambos paneles:

**App.js**
```jsx
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>Alma Ata, Kazajistán</h2>
      <Panel title="Acerca de" isActive={true}> // 👈
        Con una población de unos 2 millones de habitantes, Alma Ata es la mayor ciudad de Kazajistán. De 1929 a 1997 fue su capital.
      </Panel>
      <Panel title="Etimología" isActive={true}> // 👈
        El nombre proviene de <span lang="kk-KZ">алма</span>, palabra en kazajo que significa "manzana" y suele traducirse como "lleno de manzanas". De hecho, se cree que la región que rodea a Alma Ata es el hogar ancestral de la manzana, y se considera que este fruto silvestre <i lang="la">Malus sieversii</i> es un candidato probable para el ancestro de la manzana doméstica moderna.
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Mostrar
        </button>
      )}
    </section>
  );
}
```
Muestra:

![[3-compartir-estado-entre-componentes-3.png]]

Intenta editar los valores codificados de `isActive` en el componente `Accordion` y ve el resultado en la pantalla.

### Paso 3: Añadir el estado al componente padre común

**Elevar el estado suele cambiar la naturaleza de lo que se almacena como estado**.

En este caso, solo un panel debe estar activo a la vez. Esto significa que el componente padre común `Accordion` necesita llevar la cuenta de _qué_ panel es el que se encuentra activo. En lugar de un valor `booleano`, podría utilizar un número como índice del `Panel` activo para la variable de estado:

```jsx
const [activeIndex, setActiveIndex] = useState(0);
```

Cuando el `activeIndex` es `0`, el primer panel está activo, y cuando es `1`, lo estará el segundo.

Al hacer clic en el botón «?» en cualquiera de los dos `Paneles` es necesario cambiar el índice activo en el `Accordion`. Un `Panel` no puede establecer el estado `activeIndex` directamente porque está definido dentro del `Accordion`. El componente `Accordion` necesita _permitir explícitamente_ que el componente `Panel` cambie su estado [pasando un controlador de evento como prop](https://es.react.dev/learn/responding-to-events#passing-event-handlers-as-props):

```jsx
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)} // 👈
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)} // 👈
  >
    ...
  </Panel>
</>
```

El `<button>` dentro del `Panel` ahora usará como prop `onShow` como su controlador de evento de click:

**App.js**
```jsx
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Alma Ata, Kazajistán</h2>
      <Panel
        title="Acerca de"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        Con una población de unos 2 millones de habitantes, Alma Ata es la mayor ciudad de Kazajistán. De 1929 a 1997 fue su capital.
      </Panel>
      <Panel
        title="Etimología"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        El nombre proviene de <span lang="kk-KZ">алма</span>, palabra en kazajo que significa "manzana" y suele traducirse como "lleno de manzanas". De hecho, se cree que la región que rodea a Alma Ata es el hogar ancestral de la manzana, y se considera que este fruto silvestre <i lang="la">Malus sieversii</i> es un candidato probable para el ancestro de la manzana doméstica moderna.
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}> // 👈
          Mostrar
        </button>
      )}
    </section>
  );
}

```

¡Esto completa la elevación del estado! Mover el estado al componente padre común permitió coordinar los dos paneles. El uso de la activación por índice, en lugar de dos props «isShow», aseguró que solo un panel se encuentre activo en un momento dado. Y pasar el controlador de evento al hijo permitió al hijo cambiar el estado del padre.

![[3-compartir-estado-entre-componentes-5.png|650]]

#### Componentes controlados y no controlados

Es común llamar a un componente con algún estado local «_no controlado_». Por ejemplo, el componente original `Panel` con una variable de estado `isActive` no está controlado porque su padre no puede influir sobre el `Panel` si está activo o no.

Por el contrario, se podría decir que un componente es «_controlado_» cuando la información importante en él es determinada por props en lugar de su propio estado local. Esto permite al componente padre especificar completamente su comportamiento. El componente final de `Panel` con la propiedad `isActive` es controlado por el componente `Accordion`.

Los componentes no controlados son más fáciles de usar dentro de sus padres porque requieren menos configuración. Pero son menos flexibles cuando quieres coordinarlos entre sí. Los componentes controlados son lo más flexible, pero requieren que los componentes padres los configuren completamente con props.

En la práctica, componentes «controlado» y «no controlado» no son términos técnicos estrictos: cada componente suele tener una mezcla de estado local y por props. Sin embargo, es una forma útil de describir cómo se diseñan los componentes y qué capacidades ofrecen.

Cuando escribas un componente, plantéate qué información debe ser controlada (mediante props), y qué información debe ser no controlada (mediante estado). Pero siempre puedes cambiar de opinión y refactorizar más tarde.
## Una única fuente de verdad para cada estado

En una aplicación React, muchos componentes tendrán su propio estado. Algunos estados pueden «_vivir_» cerca de los componentes hoja (componentes en la parte inferior del árbol) como las entradas. Otros estados pueden «_vivir_» más cerca de la parte superior de la aplicación. Por ejemplo, incluso las bibliotecas de enrutamiento del lado del cliente suelen implementarse almacenando la ruta actual en el estado de React, y pasándola hacia abajo mediante props.

**Para cada estado individualizado, se elegirá el componente que lo «_albergue_». Este principio también se conoce como tener una [«única fuente de la verdad» (single source of truth).](https://en.wikipedia.org/wiki/Single_source_of_truth)** No significa que todo el estado viva en un solo lugar, sino que para _cada_ pieza de estado, hay un componente _específico_ que contiene esa pieza de información. **En lugar de duplicar el estado compartido entre los componentes, lo _elevarás_ a su padre común compartido, y lo _pasarás a los hijos_ que lo necesiten**.

Tu aplicación cambiará a medida que trabajes en ella. Es común que muevas el estado hacia abajo o hacia arriba mientras aún estás averiguando dónde «_vive_» cada pieza del estado. Todo esto forma parte del proceso.

Para ver cómo se siente esto en la práctica con algunos componentes más, lee [Pensar en React.](https://es.react.dev/learn/thinking-in-react)

## ⭐ Recapitulación

- **Cuando quieras coordinar dos componentes, mueve su estado a su padre común**.
- Luego pasa la información hacia abajo a través de props desde su padre común.
- Finalmente, pasa los controladores de eventos hacia abajo para que los hijos puedan cambiar el estado del padre.
- Es útil considerar los componentes como «controlados» (manejados por accesorios) o «no controlados» (manejados por el estado).