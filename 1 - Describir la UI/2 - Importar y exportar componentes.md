> https://es.react.dev/learn/importing-and-exporting-components

La magia de los componentes reside en su reusabilidad: puedes crear componentes que se componen a su vez de otros componentes. Pero **mientras anidas más y más componentes, a menudo tiene sentido comenzar a separarlos en diferentes archivos**. Esto permite que tus archivos se mantengan fáciles de localizar y puedas reutilizar componentes en más lugares.

### Aprenderás

- Qué es un archivo de componente raíz
- Cómo importar y exportar un componente
- Cuándo usar _imports_ y _exports_ _defaults_ o con nombre
- Cómo importar o exportar múltiples componentes de un archivo
- Cómo separar componentes en múltiples archivos

## El archivo de componente raíz

En [[1 - Tu primer componente|la lección pasada]], hiciste un componente `Profile` y un componente `Gallery` que lo renderiza:

**App.js**
```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Científicos increíbles</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

Muestra:

![[1-tu-primer-componente-2.png]]

**Estos componentes viven actualmente en este ejemplo en un _archivo de componente raíz_, llamado `App.js`**. No obstante, dependiendo de tu configuración, tu componente raíz podría estar en otro archivo. Si utilizas un framework con enrutamiento basado en archivos, como Next.js, tu componente raíz será diferente para cada página.

## Exportar e importar un componente

¿Y si quisieras cambiar la pantalla de inicio en el futuro y poner allí una lista de libros científicos? ¿O ubicar todos los perfiles en otro lugar? **Tiene sentido mover `Gallery` y `Profile` fuera del componente raíz**. Esto los haría más modulares y reutilizables en otros archivos. Puedes mover un componente en tres pasos:

1. **Crea** un nuevo archivo JS para poner los componentes dentro.
2. **Exporta** tu componente de función desde ese archivo (ya sea usando exports [por defecto](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Statements/export#usando_el_export_por_defecto) o [con nombre](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Statements/export#syntax)).
3. **Impórtalo** en el archivo en el que usarás el componente (usando la técnica correspondiente de importar exports [por defecto](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Statements/import#importaci%C3%B3n_de_elementos_por_defecto) o [con nombre](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Statements/import#importa_un_solo_miembro_de_un_m%C3%B3dulo.)).

Aquí tanto `Profile` y `Gallery` se han movido fuera de `App.js` en un nuevo archivo llamado `Gallery.js`. Ahora puedes cambiar `App.js` para importar `Gallery` desde `Gallery.js`:

**App.js**
```jsx
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}
```
**Gallery.js**
```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Científicos increíbles</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```


Nota cómo este ejemplo está ahora descompuesto en dos archivos:

1. `Gallery.js`:
    - Define el componente `Profile` que se usa solo dentro del mismo archivo y no se exporta.
    - Define el componente `Gallery` como un **export por defecto**.
2. `App.js`:
    - Importa `Gallery` como un **import por defecto** desde `Gallery.js`.
    - Exporta el componente raíz `App` como un **export por defecto**.

>[!note]
>Puede que te encuentres archivos que omiten la extensión de archivo `.js` de esta forma:
>
>```jsx
>import Gallery from './Gallery';
>```
>
>Tanto `'./Gallery.js'` como `'./Gallery'` funcionarán con React, aunque la primera forma es más cercana a cómo lo hacen los [módulos nativos de ES](https://developer.mozilla.org/es/docs/Web/JavaScript/Guide/Modules).

#### Exports por defecto vs. con nombre 

![[2-importar-y-expoertar-componentes-1.svg]]

Cómo exportas tu componente dicta la forma en que debes importarlo. ¡Tendrás un error si intentas importar un export por defecto de la misma forma que lo harías con un export con nombre! Este cuadro te puede ayudar a recordarlo:

| Sintaxis    | Sentencia export                      | Sentencia import                        |
| ----------- | ------------------------------------- | --------------------------------------- |
| Por defecto | `export default function Button() {}` | `import Button from './Button.js';`     |
| Con nombre  | `export function Button() {}`         | `import { Button } from './Button.js';` |

Cuando escribes un import _por defecto_ puedes poner cualquier nombre después de `import`. Por ejemplo, podrías escribir en su lugar `import Banana from './Button.js'` y aun así te daría el mismo export por defecto. En cambio, con los imports con nombre, tiene que haber una correspondencia con los nombres en ambos lados. ¡Por eso se llaman exports _con nombre_!

**Las personas a menudo utilizan exports por defecto si el archivo solo exporta un componente, y usan exports con nombre si exporta varios componentes y valores.** Independientemente del estilo de codificación que prefieras, siempre proporciona nombres con sentido a las funciones de tus componentes y a los archivos que las contienen. Componentes sin nombre como `export default () => {}` no se recomiendan, porque hacen que la depuración sea más difícil.

## Exportar e importar múltiples componentes del mismo archivo 

¿Y si quisieras mostrar solo un `Profile` en lugar de toda la galería? Puedes exportar el componente `Profile` también. Pero `Gallery.js` ya tiene un export _por defecto_, y no puedes tener _dos_ exports por defecto. Podrías crear un nuevo archivo con un export por defecto, o podrías añadir un export _con nombre_ para `Profile`. **¡Un archivo solo puede contener un export por defecto, pero puede tener múltiples exports con nombre!**

> [!note]
> Para reducir la potencial confusión entre _exports_ por defecto y con nombre, algunos equipos escogen utilizar solo un estilo (por defecto o con nombre), o evitan mezclarlos en un mismo archivo. Es una cuestión de preferencias. ¡Haz lo que funcione mejor para ti!

Primero, **exporta** `Profile` desde `Gallery.js` usando un `export` con nombre (sin la palabra clave `default`):

```jsx
export function Profile() {
  // ...
}
```

Luego, **importa** `Profile` de `Gallery.js` a `App.js` usando un `import` con nombre (con llaves):

```jsx
import { Profile } from './Gallery.js';
```

Por último, **renderiza** `<Profile />` en el componente `App`:

```jsx
export default function App() {
  return <Profile />;
}
```

Ahora `Gallery.js` contiene dos exports: un export por defecto `Gallery`, y un export con nombre `Profile`. `App.js` importa ambos. Intenta editar `<Profile />` cambiándolo a `<Gallery />` y viceversa en este ejemplo:

**App.js**
```jsx
import Gallery from './Gallery.js';
import { Profile } from './Gallery.js';

export default function App() {
  return (
    <Profile />
  );
}

```
**Gallery.js**
```jsx
export function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Científicos increíbles</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}

```

Ahora estás usando una mezcla de exports por defecto y con nombre:

- `Gallery.js`:
    - Exporta el componente `Profile` como un **export con nombre llamado `Profile`**.
    - Exporta el componente `Gallery` como un **export por defecto**.
- `App.js`:
    - Importa `Profile` como un **import con nombre llamado `Profile`** desde `Gallery.js`.
    - Importa `Gallery` como un **import por defecto** desde `Gallery.js`.
    - Exporta el componente raíz `App` como un **export por defecto**.

> [!important] ⭐
> Si bien vemos que es posible tener más de un componente en un mismo archivo, **lo recomendable** es siempre tener un sólo archivo por componente, esto ayuda a la mantenibilidad, especialmente en proyectos grandes
## Recapitulación

En esta página aprendiste:

- Qué es un archivo de componente raíz
- Como importar y exportar un componente
- Cuándo y cómo usar _imports_ y _exports_ por defecto y con nombre
- Cómo exportar múltiples componentes desde el mismo archivo
