El estado puede contener cualquier tipo de valor JavaScript, incluyendo objetos. Pero **no deberías cambiar los objetos que tienes en el estado de React directamente. En su lugar, cuando quieras actualizar un objeto, tienes que crear uno nuevo (o hacer una copia de uno existente), y luego configurar el estado para usar esa copia**.

### Aprenderás

- Cómo actualizar correctamente un objeto en el estado de React
- Cómo actualizar un objeto anidado sin mutarlo
- Qué es la inmutabilidad y cómo no romperla
- Cómo hacer que la copia de objetos sea menos repetitiva con Immer

## ⭐¿Qué es una mutación?

Puede almacenar cualquier tipo de valor de JavaScript en el estado.

```jsx
const [x, setX] = useState(0);
```

Hasta ahora has trabajado con números, _strings_ y booleanos. Estos tipos de valores de JavaScript son «_inmutables_», es decir, inmutables o de «_sólo lectura_». Se puede activar un re-renderizado para _reemplazar_ un valor:

```jsx
setX(5);
```

**El estado `x` ha cambiado de `0` a `5`, pero el _número `0` en sí mismo_ no cambia**. No es posible hacer ningún cambio en los valores primitivos incorporados como números, _strings_ y booleanos en JavaScript.

Consideremos ahora un objeto en estado:

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Técnicamente, es posible cambiar el contenido del _objeto mismo_. **Esto se denomina mutación:**.

```jsx
position.x = 5;
```

Sin embargo, **aunque los objetos en el estado de React son técnicamente mutables, deberías tratarlos como si fueran inmutables—como los números, booleanos y _strings_. En lugar de mutarlos, siempre debes reemplazarlos**.

## Tratar el estado como de sólo lectura

En otras palabras, **debes tratar cualquier objeto JavaScript que pongas en estado como de sólo lectura.**

Este ejemplo mantiene un objeto en el estado para representar la posición actual del puntero. Se supone que el punto rojo se mueve cuando se toca o se mueve el cursor, sobre el área de vista previa. Pero el punto permanece en la posición inicial:

**App.js**
```jsx
import { useState } from 'react';
export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        position.x = e.clientX;
        position.y = e.clientY;
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  );
}
```

Muestra:

![[6-actualiza-objetos-en-el-estado-1.png]]

El problema está en este trozo de código.

```jsx
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```

**Este código modifica el objeto asignado a `position` desde [el renderizado anterior.](https://es.react.dev/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) Pero sin usar la función de ajuste de estado, React no tiene idea de que el objeto ha cambiado. Así que React no hace nada en respuesta**. Es como intentar cambiar el orden de lo que has comido cuando ya has acabado. 

>[!danger]
>Aunque mutar el estado puede funcionar en algunos casos, no lo recomendamos. Debes tratar el valor del estado al que tienes acceso en un renderizado como de sólo lectura.

Para realmente [conseguir un re-renderizado](https://es.react.dev/learn/state-as-a-snapshot#setting-state-triggers-renders) en este caso, **crea un objeto _nuevo_ y pásalo a la función de configuración de estado:**

```jsx
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

Con `setPosition`, le estás diciendo a React:

- Reemplaza `position` con este nuevo objeto
- Y renderiza este componente de nuevo

Observa cómo el punto rojo sigue ahora a tu puntero cuando tocas o pasas el ratón por encima del área de vista previa:

**App.js**
```jsx
import { useState } from 'react';
export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        setPosition({ // 👈✅
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  );
}
```

Muestra:

![[6-actualiza-objetos-en-el-estado-2.png]]

#### La mutación local es correcta

Un código como este es un problema porque modifica un objeto _existente_ en el estado:

```jsx
position.x = e.clientX;
position.y = e.clientY;
```

Pero un código como el siguiente está **absolutamente bien**, ya que estarías mutando un objeto que _acabas de crear_:

```jsx
const nextPosition = {};
nextPosition.x = e.clientX;
nextPosition.y = e.clientY;
setPosition(nextPosition);
```

Esto está bien por que es completamente equivalente a escribir lo siguiente:

```jsx
setPosition({
  x: e.clientX,
  y: e.clientY
});
```

**La mutación sólo es un problema cuando cambias objetos _existentes_ que ya están en el estado**. Mutar un objeto que acabas de crear está bien porque _ningún otro código hace referencia a él todavía._ Cambiarlo no va a afectar accidentalmente a algo que dependa de él. Esto se llama «_mutación local_». Incluso puedes hacer una mutación local [mientras renderizas.](https://es.react.dev/learn/keeping-components-pure#local-mutation-your-components-little-secret) ¡Muy conveniente y completamente bien!

## ⭐ Copiar objetos con la sintaxis extendida

En el ejemplo anterior, el objeto `position` se crea siempre de nuevo a partir de la posición actual del cursor. Pero **a menudo, querrás incluir datos _existentes_ como parte del nuevo objeto que estás creando. Por ejemplo, puedes querer actualizar _sólo un_ campo de un formulario, pero mantener los valores anteriores de todos los demás campos**.

Estos campos de entrada no funcionan porque los controladores `onChange` mutan el estado:

**App.js**
```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    person.firstName = e.target.value;
  }

  function handleLastNameChange(e) {
    person.lastName = e.target.value;
  }

  function handleEmailChange(e) {
    person.email = e.target.value;
  }

  return (
    <>
      <label>
        Nombre:
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Apellido:
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        Correo electrónico:
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

Muestra: 

![[6-actualiza-objetos-en-el-estado-3.png]]

Por ejemplo, esta línea muta el estado de un render pasado:

```jsx
person.firstName = e.target.value;
```

La forma fiable de obtener el comportamiento que buscas es crear un nuevo objeto y pasarlo a `setPerson`. Pero aquí, quieres también **copiar los datos existentes en él** porque sólo uno de los campos ha cambiado:

```jsx
setPerson({
  firstName: e.target.value, // New first name from the input
  lastName: person.lastName,
  email: person.email
});
```

**Para ello, se puede utilizar el `...` [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals) para no tener que copiar cada propiedad por separado**.

```jsx
setPerson({
  ...person, // Copy the old fields
  firstName: e.target.value // But override this one
});
```

¡Ahora el formulario funciona!

Fíjate en que no has declarado una variable de estado distinta para cada campo de entrada. Para los formularios grandes, es muy conveniente mantener todos los datos agrupados - ¡siempre y cuando los actualices correctamente!

**App.js**
```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    setPerson({
      ...person, // 👈
      firstName: e.target.value
    });
  }

  function handleLastNameChange(e) {
    setPerson({
      ...person, // 👈
      lastName: e.target.value
    });
  }

  function handleEmailChange(e) {
    setPerson({
      ...person, // 👈
      email: e.target.value
    });
  }

  return (
    <>
      <label>
        Nombre:
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Apellido:
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        Correo electrónico:
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

Muestra:

![[6-actualiza-objetos-en-el-estado-3.png]]

**Ten en cuenta que la sintaxis extendida `...` es «_superficial_»: sólo copia las cosas a un nivel de profundidad. Esto lo hace rápido, pero también significa que si quieres actualizar una propiedad anidada, tendrás que usarla más de una vez**.

#### ⭐ Utilizar un único controlador de evento para diversos campos

También puedes utilizar las llaves `[` y `]` dentro de tu definición de objeto para especificar una propiedad con nombre dinámico. Aquí está el mismo ejemplo, pero con un solo controlador de evento en lugar de tres diferentes:

**App.js**
```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value
    });
  }

  return (
    <>
      <label>
        Nombre:
        <input
          name="firstName"
          value={person.firstName}
          onChange={handleChange}
        />
      </label>
      <label>
        Apellido:
        <input
          name="lastName"
          value={person.lastName}
          onChange={handleChange}
        />
      </label>
      <label>
        Correo electrónico:
        <input
          name="email"
          value={person.email}
          onChange={handleChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

Aquí, `e.target.name` se refiere a la propiedad `name` dada al elemento DOM `<input>`.

## ⭐ Actualizar un objeto anidado

Considera una estructura de objetos anidados como esta:

```jsx
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Nana azul',
    city: 'Hamburgo',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});
```

Si quisieras actualizar `person.artwork.city`, está claro cómo hacerlo con la mutación:

```jsx
person.artwork.city = 'Nueva Delhi';
```

Pero en React, ¡se trata el estado como inmutable! **Para cambiar la «_ciudad_», primero tendrías que producir el nuevo objeto «_artwork_» (pre-poblado con los datos de la anterior), y luego producir el nuevo objeto «_person_» que apunta a la nueva «_artwork_»**:

```jsx
const nextArtwork = { 
  ...person.artwork, 
  city: 'Nueva Delhi' 
};
const nextPerson = { 
  ...person, 
  artwork: nextArtwork
};
setPerson(nextPerson);
```

O, escrito como una sola llamada a la función:

```jsx
setPerson({
  ...person, // Copia otros campos
  artwork: { // pero sustituye el artwork
    ...person.artwork, // por el mismo
    city: 'Nueva Delhi' // ¡pero en Nueva Delhi!
  }
});
```

Esto es un poco complicado, pero funciona bien para muchos casos:

**App.js**
```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Nana azul',
      city: 'Hamburgo',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value
      }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value
      }
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value
      }
    });
  }

  return (
    <>
      <label>
        Nombre:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Título:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        Ciudad:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Imagen:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' por '}
        {person.name}
        <br />
        (situada en {person.artwork.city})
      </p>
      <img 
        src={person.artwork.image} 
        alt={person.artwork.title}
      />
    </>
  );
}
```

Muestra:

![[6-actualiza-objetos-en-el-estado-4.png]]

#### Los objetos no están realmente anidados

Un objeto de este tipo aparece «anidado» en el código:

```jsx
let obj = {
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Nana azul',
    city: 'Hamburgo',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
};
```

**Sin embargo, la «_anidación_» es una forma inexacta de pensar en el comportamiento de los objetos. Cuando el código se ejecuta, no existe tal cosa como un objeto «anidado». ==En realidad, se trata de dos objetos diferentes==**:

```jsx
let obj1 = {
  title: 'Nana azul',
  city: 'Hamburgo',
  image: 'https://i.imgur.com/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};
```

El objeto `obj1` no está «_dentro_» de `obj2`. Por ejemplo, `obj3` también podría «_apuntar_» a `obj1`:

```jsx
let obj1 = {
  title: 'Nana azul',
  city: 'Hamburgo',
  image: 'https://i.imgur.com/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};

let obj3 = {
  name: 'Copycat',
  artwork: obj1
};
```

Si se muta `obj3.artwork.city`, afectaría tanto a `obj2.artwork.city` como a `obj1.city`. Esto se debe a que `obj3.artwork`, `obj2.artwork` y `obj1` son el mismo objeto. Esto es difícil de ver cuando se piensa en los objetos como «anidados». En cambio, **son objetos separados que se «_apuntan_» unos a otros con propiedades**.

### ⭐ Escribe una lógica de actualización concisa con Immer

Si su estado está profundamente anidado, podría considerar [aplanarlo.](https://es.react.dev/learn/choosing-the-state-structure#avoid-deeply-nested-state) Pero, si no quieres cambiar la estructura de tu estado, puede que prefieras un atajo a los spreads anidados. **[Immer](https://github.com/immerjs/use-immer) es una popular librería que te permite escribir utilizando la sintaxis conveniente pero mutante y se encarga de producir las copias por ti. Con Immer, el código que escribes parece que estés «_rompiendo las reglas_» y mutando un objeto**:

```jsx
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

Pero a diferencia de una mutación normal, ¡no sobrescribe el estado anterior!

> [!info]
>#### ¿Cómo funciona Immer?
>
>El borrador (`draft`) proporcionado por Immer es un tipo especial de objeto, llamado [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) (el mismo que usa Vue para su reactividad), que «_registra_» lo que haces con él. Por eso, ¡puedes mutar libremente todo lo que quieras! Bajo el capó, Immer se da cuenta de qué partes del «borrador» han sido cambiadas, y produce un objeto completamente nuevo que contiene tus ediciones.

Para probar Immer:

1. Ejecuta `npm install use-immer` para añadir Immer como dependencia
2. Luego sustituye `import { useState } de 'react'` por `import { useImmer } de 'use-immer'`.

Aquí está el ejemplo anterior convertido a Immer:

**App.jsx**
```jsx
import { useImmer } from 'use-immer';

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Nana azul',
      city: 'Hamburgo',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => { // 👈
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson(draft => { // 👈
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson(draft => { // 👈
      draft.artwork.city = e.target.value;
    });
  }

  function handleImageChange(e) {
    updatePerson(draft => { // 👈
      draft.artwork.image = e.target.value;
    });
  }

  return (
    <>
      <label>
        Nombre:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Título:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        Ciudad:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Imagen:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' por '}
        {person.name}
        <br />
        (situada en {person.artwork.city})
      </p>
      <img 
        src={person.artwork.image} 
        alt={person.artwork.title}
      />
    </>
  );
}
```

**package.json**
```jsx
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "devDependencies": {}
}
```

Muestra:

![[6-actualiza-objetos-en-el-estado-4.png]]

Fíjate en lo mucho más concisos que se han vuelto los controladores de eventos. Puedes mezclar y combinar `useState` y `useImmer` en un mismo componente tanto como quieras. Immer es una gran manera de mantener los controladores de actualización de manera concisa, especialmente si hay anidación en su estado, y la copia de objetos conduce a código repetitivo.

#### ¿Por qué no se recomienda mutar el estado en React?

Hay algunas razones:

- **Debugging:** Si usas `console.log` y no mutas el estado, tus registros anteriores no se verán afectados por los cambios de estado más recientes. Así puedes ver claramente cómo ha cambiado el estado entre renders.
- **Optimizaciones:** Las [estrategias de optimización](https://es.react.dev/reference/react/memo) más comunes en React se basan en ahorrar trabajo si las props o el estado anteriores son los mismos que los siguientes. Si nunca se muta el estado, es muy rápido comprobar si ha habido algún cambio. Si `prevObj === obj`, puedes estar seguro de que nada ha podido cambiar en su interior.
- **Nuevas características:** Las nuevas características de React que estamos construyendo dependen de que el estado sea [tratado como una instantánea.](https://es.react.dev/learn/state-as-a-snapshot) Si estás mutando versiones anteriores del estado, eso puede impedirte utilizar las nuevas funciones.
- **Cambios de requisitos:** Algunas características de la aplicación, como la implementación de Deshacer/Rehacer, mostrar un historial de cambios, o permitir al usuario reiniciar un formulario a valores anteriores, son más fáciles de hacer cuando no se muta nada. Esto se debe a que puedes mantener copias pasadas del estado en la memoria, y reutilizarlas cuando sea apropiado. Si empiezas con un enfoque mutativo, características como estas pueden ser difíciles de añadir más adelante.
- **Implementación más sencilla:** Como React no se basa en la mutación, no necesita hacer nada especial con tus objetos. No necesita apropiarse de sus propiedades, envolverlos siempre en Proxies, o hacer otro trabajo en la inicialización como hacen muchas soluciones «reactivas». Esta es también la razón por la que React te permite poner cualquier objeto en el estado - no importa lo grande que sea - sin problemas adicionales de rendimiento o corrección.

En la práctica, a menudo puedes «salirte con la tuya» con la mutación de estado en React, pero te aconsejamos encarecidamente que no lo hagas para que puedas utilizar las nuevas características de React desarrolladas con este enfoque en mente. Los futuros colaboradores y quizás incluso tú mismo en el futuro te lo agradecerán.

## Recapitulación

- Tratar todo el estado en React como inmutable.
- Cuando se almacenan objetos en el estado, la mutación de los mismos no desencadenará renderizados y cambiará el estado en las «instantáneas» de los renderizados anteriores.
- En lugar de mutar un objeto, crea una _nueva_ versión del mismo, y dispara un re-renderizado estableciendo el estado en él.
- Puedes usar la sintaxis extendida de objetos `{...obj, algo: 'newValue'}` para crear copias de objetos.
- La sintaxis extendida es superficial: sólo copia un nivel de profundidad.
- Para actualizar un objeto anidado, necesitas crear copias desde el lugar que estás actualizando.
- Para reducir el código de copia repetitivo, utiliza Immer.