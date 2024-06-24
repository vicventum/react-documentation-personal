Algunas funciones de JavaScript son _puras._ Las funciones puras solo realizan un cálculo y nada más. Al escribir estrictamente tus componentes como funciones puras, puedes evitar una clase completa de errores desconcertantes y un comportamiento impredecible a medida que crece tu base de código. Sin embargo, para obtener estos beneficios, hay algunas reglas que debes seguir.

### Aprenderás

- Qué es la pureza y cómo te ayuda a evitar errores
- Cómo mantener los componentes puros manteniendo los cambios fuera de la fase de renderizado
- Cómo usar el modo estricto para encontrar errores en tus componentes

## Pureza: componentes como fórmulas

En informática (y especialmente en el mundo de la programación funcional), [una función pura](https://wikipedia.org/wiki/Pure_function) es una función con las siguientes características:

- **Se ocupa de sus propios asuntos.** No cambia ningún objeto o variable que existiera antes de ser llamado.
- **Las mismas entradas, la misma salida.** Dadas las mismas entradas, una función pura siempre debe devolver el mismo resultado.

Es posible que ya estés familiarizado con un ejemplo de funciones puras: fórmulas en matemáticas.

Considera esta fórmula matemática: $y = 2x$.

Si $x = 2$ entonces $y = 4$. Siempre.

Si $x = 3$ entonces $y = 6$. Siempre.

Si $x = 3$, y a veces no será $9$ o $–1$ o $2.5$ dependiendo de la hora del día o del estado del mercado de valores.

Si $y = 2x$ y $x = 3$, y _siempre_ será $6$.

Si convirtiéramos esto en una función de JavaScript, se vería así:

```jsx
function double(number) {
  return 2 * number;
}
```

En el ejemplo anterior, `double` es una **función pura.** Si le pasas `3`, devolverá `6`. Siempre.

React está diseñado en torno a este concepto. **React supone que cada componente que escribes es una función pura.** Esto significa que los componentes que escribes en React siempre deben devolver el mismo JSX dadas las mismas entradas:

**App.js**
```jsx
function Recipe({ drinkers }) {
  return (
    <ol>    
      <li>Hervir {drinkers} tazas de agua.</li>
      <li>Añadir {drinkers} cucharadas de té y {0.5 * drinkers} cucharada de especias.</li>
      <li>Añadir {0.5 * drinkers} tazas de leche hirviendo y azúcar a gusto.</li>
    </ol>
  );
}

export default function App() {
  return (
    <section>
      <h1>Receta de té Chai especiado</h1>
      <h2>Para dos</h2>
      <Recipe drinkers={2} />
      <h2>Para una reunión</h2>
      <Recipe drinkers={4} />
    </section>
  );
}
```

Cuando pasas `drinkers={2}` a `Recipe`, devolverá el JSX que contiene `2 cups of water`. Siempre.

Si pasas `drinkers={4}`, devolverá el JSX que contiene `4 cups of water`. Siempre.

Como una fórmula matemática.

Puedes pensar en tus componentes como recetas: si las sigues y no agregas nuevos ingredientes durante el proceso de cocción, obtendrás el mismo plato siempre. Ese «plato» es el JSX que el componente le pasa a React para [renderizar.](https://es.react.dev/learn/render-and-commit)

![Una receta de té para x personas: toma x tazas de agua, añade x cucharadas de té y 0.5x cucharadas de especias y 0.5x tazas de leche](https://es.react.dev/images/docs/illustrations/i_puritea-recipe.png)

Ilustrado por [Rachel Lee Nabors](https://nearestnabors.com/)

## Efectos secundarios (side effects): consecuencias (no) deseadas

El proceso de renderizado de React siempre debe ser puro. **Los componentes solo deben _devolver_ su JSX, y no _cambiar_ cualquier objeto o variable que existiera antes de renderizar**: ¡Eso los haría impuros!

Aquí hay un componente que rompe esta regla:

**App.js**
```jsx
let guest = 0; // 👈👎

function Cup() {
  // Mal: ¡Cambiar una variable preexistente!
  guest = guest + 1; // 👈👎
  return <h2>Taza de té para invitado #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

Este componente está leyendo y escribiendo una variable `guest` declarada fuera de ella. Esto significa que **llamar a este componente varias veces producirá diferente JSX!** Y lo que es más, si _otros_ componentes leen `guest`, también producirán diferente JSX, ¡dependiendo de cuándo se procesaron! Eso no es predecible.

Volviendo a nuestra fórmula $y = 2x$, ahora incluso si $x = 2$, no podemos confiar en que $y = 4$. Nuestras pruebas podrían fallar, nuestros usuarios estarían desconcertados, los aviones se caerían del cielo —¡puedes ver cómo esto conduciría a errores confusos!

Puedes arreglar este componente [pasando `guest` como prop en su lugar](https://es.react.dev/learn/passing-props-to-a-component):

**App.js**
```jsx
function Cup({ guest }) { // 👈👍
  return <h2>Taza de té para invitado #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} /> // 👈👍
      <Cup guest={2} /> // 👈👍
      <Cup guest={3} /> // 👈👍
    </>
  );
}
```

**Ahora tu componente ya es puro, ya que el JSX que devuelve solo depende de la prop `guest`**.

En general, **no debes esperar que tus componentes se rendericen en ningún orden en particular**. No importa si llamas $y = 2x$ antes o después $y = 5x$: ambas fórmulas se resolverán independientemente una de la otra. Del mismo modo, cada componente solo debe «pensar por sí mismo» y no intentar coordinarse o depender de otros durante el renderizado. El renderizado es como un examen escolar: ¡cada componente debe calcular su JSX por su cuenta!

### ⭐ Detección de cálculos impuros con _Strict Mode_

Aunque es posible que aún no los hayas usado todos, en React **hay tres tipos de entradas que puedes leer mientras se renderiza: [props](https://es.react.dev/learn/passing-props-to-a-component), [state](https://es.react.dev/learn/state-a-components-memory), y [context.](https://es.react.dev/learn/passing-data-deeply-with-context) Siempre debes tratar estas entradas como solo lectura**.

**Cuando quieras _cambiar_ algo en respuesta a la entrada del usuario, debes [asignar el estado](https://es.react.dev/learn/state-a-components-memory) en lugar de reescribir la variable**. Nunca debes cambiar variables u objetos preexistentes mientras tu componente está renderizando.

React ofrece un «_Modo estricto_» en el que llama a la función de cada componente dos veces durante el desarrollo. **Al llamar a las funciones del componente dos veces, el modo estricto ayuda a encontrar componentes que rompan estas reglas.**

Observa cómo el ejemplo original mostraba «`Guest #2`», «`Guest #4`», y «`Guest #6`» en lugar de «`Guest #1`», «`Guest #2`», y «`Guest #3`». **La función original era impura, por lo que al llamarla dos veces se rompió**. Pero la versión corregida funciona sin importar que la función sea llamada dos veces cada vez. **Las funciones puras solo se calculan, por lo que llamarlas dos veces no cambiará nada** —como llamar `double(2)` dos veces no cambia lo que se devuelve, y devuelve y = 2x dos veces, no cambia lo que y es. Las mismas entradas, las mismas salidas. Siempre.

El modo estricto no tiene ningún efecto en producción, por lo que no ralentizará la aplicación para tus usuarios. Para optar por el modo estricto, puedes envolver tu componente raíz en `<React.StrictMode>`. Algunos frameworks hacen esto por defecto.

### Mutación local: el pequeño secreto de tus componentes

**En el ejemplo anterior, el problema era que el componente cambiaba una variable _preexistente_ mientras renderizaba. Esto a menudo se llama «_mutación_»** para que suene un poco más aterrador. **¡Las funciones puras no mutan las variables fuera del alcance de la función ni los objetos que se crearon antes de la llamada —¡Eso las hace impuras**!

Sin embargo, **está completamente bien cambiar variables y objetos que acabas de crear mientras renderizas.** En este ejemplo, creas un _array_ `[]`, lo asignas a la variable `cups`, y luego haces un `push` con una docena de tazas adentro:

**App.js**
```jsx
function Cup({ guest }) {
  return <h2>Taza de té para invitado #{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

¡Si la variable `cups` o el _array_ `[]` se crearon fuera de la función `TeaGathering`, este sería un gran problema! Estarías cambiando un objeto _preexistente_ haciendo push a ese array.

Sin embargo, está bien porque los has creado _durante el mismo renderizado_, dentro de `TeaGathering`. Ningún código fuera de `TeaGathering` sabrá nunca que esto ha ocurrido. Esto se llama **«mutación local»** —es como el pequeño secreto de tu componente.

## ¿Dónde _puedes_ causar efectos secundarios?

Si bien la programación funcional depende en gran medida de la pureza, en algún momento, en algún lugar, _algo_ tiene que cambiar. ¡Ese es el punto en programación! Estos cambios —actualizar la pantalla, iniciar una animación, cambiar los datos— se llaman **efectos secundarios (o _side effects_ en inglés).** Son cosas que suceden _«a un lado»_, no durante el renderizado.

En React, **los efectos secundarios generalmente deberían estar dentro de los [controladores de eventos.](https://es.react.dev/learn/responding-to-events)** Los controladores de eventos son funciones que React ejecuta cuando realiza alguna acción (por ejemplo, cuando haces clic en un botón). **¡Aunque los controladores de eventos están definidos _dentro_ de tu componente, no corren _durante_ el renderizado! Por lo tanto, los controladores de eventos no necesitan ser puros.**

Si has agotado todas las demás opciones y no puedes encontrar el controlador de evento adecuado para tu efecto secundario, aún puedes adjuntarlo en la devolución del JSX con un llamado a [`useEffect`](https://es.react.dev/reference/react/useEffect) en tu componente. Esto le dice a React que lo ejecute más tarde, después del renderizado, cuando se permiten efectos secundarios. **Sin embargo, este enfoque debería ser tu último recurso.**

Cuando sea posible, intenta expresar tu lógica con un solo renderizado. ¡Te sorprenderá lo lejos que esto puede llevarte!

### ¿Por qué a React le importa la pureza?

Escribir funciones puras requiere cierto hábito y disciplina. Pero también desbloquea maravillosas oportunidades:

- ¡Tus componentes podrían ejecutarse en un entorno diferente (por ejemplo, en el servidor)! Como devuelven el mismo resultado para las mismas entradas, un componente puede atender muchas solicitudes de los usuarios.
- Puedes mejorar el rendimiento [omitiendo el renderizado](https://es.react.dev/reference/react/memo) de componentes cuyas entradas no han cambiado. Esto es seguro porque las funciones puras siempre devuelven los mismos resultados, por lo que son seguras para almacenar en caché.
- Si algunos datos cambian en medio del renderizado de un árbol de componentes profundos, React puede reiniciar el renderizado sin perder tiempo para terminar el renderizado desactualizado. La pureza hace que sea seguro dejar de calcular en cualquier momento.

Cada nueva característica de React que estamos construyendo aprovecha la pureza. Desde la búsqueda de datos hasta las animaciones y el rendimiento, mantener los componentes puros desbloquea el poder del paradigma de React.

## Recapitulación

- Lo que significa que un componente debe ser puro:
    - **Se ocupa de sus propios asuntos.** No debe cambiar ningún objeto o variable que existiera antes del renderizado.
    - **Las mismas entradas, la misma salida.** Dadas las mismas entradas, un componente siempre debe devolver el mismo JSX.
- El renderizado puede ocurrir en cualquier momento, por lo que los componentes no deben depender de la secuencia de renderizado de los demás.
- No debe mutar ninguna de las entradas que usan sus componentes para renderizar. Eso incluye props, estado y contexto. Para actualizar la pantalla, [«asignar» el estado](https://es.react.dev/learn/state-a-components-memory) en lugar de mutar objetos preexistentes.
- Esfuérzate por expresar la lógica de tu componente en el JSX. Cuando necesites «cambiar cosas», generalmente querrás hacerlo en un controlador de evento. Como último recurso, puedes usar `useEffect`.
- Escribir funciones puras requiere un poco de práctica, pero desbloquea el poder del paradigma de React.