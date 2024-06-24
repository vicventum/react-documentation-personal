JSX te permite escribir marcas similares a HTML dentro de un archivo JavaScript, manteniendo la lógica de renderizado y el contenido en el mismo lugar. A veces vas a querer agregar un poco de lógica JavaScript o hacer referencia a una propiedad dinámica dentro de ese marcado. En esta situación, puedes usar llaves en tu JSX para abrir una ventana a JavaScript.

### Aprenderás

- Cómo pasar _strings_ con comillas
- Cómo hacer referencia a una variable de JavaScript dentro de JSX con llaves
- Cómo llamar una función de JavaScript dentro de JSX con llaves
- Cómo usar un objeto de JavaScript dentro de JSX con llaves

## Pasando strings con comillas

Cuando desees pasar un atributo string a JSX, lo pones entre comillas simples o dobles.

Aquí, `"https://i.imgur.com/7vQD0fPs.jpg"` y `"Gregorio Y. Zara"` están siendo pasados 
como strings:

**App.js**
```jsx
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/7vQD0fPs.jpg" 👈
      alt="Gregorio Y. Zara" 👈
    />
  );
}
```

Pero, ¿qué sucede si quieres especificar dinámicamente el texto `src` o `alt`? Puedes **usar un valor de JavaScript reemplazando las comillas de apertura `"` y de cierre `"` con las llaves de apertura `{` y de cierre `}`**:

**App.js**
```jsx
export default function Avatar() {
  const avatar = 'https://i.imgur.com/7vQD0fPs.jpg';
  const description = 'Gregorio Y. Zara';
  return (
    <img
      className="avatar"
      src={avatar} 👈
      alt={description} 👈
    />
  );
}
```

Observa la diferencia entre `className="avatar"`, que especifica un nombre de clase CSS `"avatar"` que hace que la imagen sea redonda, y **`src={avatar}` que lee el valor de una variable JavaScript llamada `avatar`**. ¡Eso es porque las llaves te permiten trabajar con JavaScript allí mismo en tu marcado!.

## ⭐ Usando llaves: Una ventana al mundo de JavaScript

JSX es una forma especial de escribir JavaScript. Eso significa que es posible utilizar JavaScript dentro de él, con llaves `{ }`. El ejemplo siguiente declara primero un nombre para el científico, `name`, y luego lo inserta con llaves dentro de `<h1>`:

**App.js**
```jsx
export default function TodoList() {
  const name = 'Hedy Lamarr';
  return (
    <h1>Lista de tareas pendientes de {name}</h1> 👈
  );
}
```  

Intenta cambiar el valor de `name` de `'Gregorio Y. Zara'` a `'Hedy Lamarr'`. ¿Ves cómo cambia el título de la lista de tareas pendientes?

Cualquier expresión JavaScript funcionará entre llaves, incluidas las llamadas a funciones como `formatDate()`:

**App.js**
```jsx
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat(
    'es-ES',
    { weekday: 'long' }
  ).format(date);
}

export default function TodoList() {
  return (
    <h1>Lista de tareas pendientes del {formatDate(today)}</h1> 👈
  );
}
```

### ⭐ Dónde usar llaves

Solo puedes usar llaves de dos maneras dentro de JSX:

1. **Como texto** directamente dentro de una etiqueta JSX: `<h1>Lista de tareas pendientes de {name}</h1>` funcionará, pero `<{tag}>Lista de tareas pendientes de Gregorio Y. Zara</{tag}>` no lo hará.
2. **Como atributos** inmediatamente después del signo `=`: `src={avatar}` leerá la variable `avatar`, pero `src="{avatar}"` pasará el string `"{avatar}"`.

## ⭐ Usando «llaves dobles»: CSS y otros objetos en JSX (etiqueta `style`)

Además de _strings_, números, y otras expresiones de JavaScript, incluso puedes pasar objetos en JSX. Los objetos también se indican con llaves, como `{ name: "Hedy Lamarr", inventions: 5 }`. Por lo tanto, para pasar un objeto de JavaScript en JSX, debes envolver el objeto en otro par de llaves: `person={{ name: "Hedy Lamarr", inventions: 5 }}`.

Puedes ver esto con estilos en línea CSS, en JSX. React no requiere que uses estilos en línea (las clases CSS funcionan muy bien para la mayoría de los casos). Pero cuando necesitas un estilo en línea, pasas un objeto al atributo `style`:

**App.js**
```jsx
export default function TodoList() {
  return (
    <ul style={{ 👈
      backgroundColor: 'salmon',
      color: 'white'
    }}>
      <li>Mejorar el videoteléfono</li>
      <li>Preparar clases de aeronáutica</li>
      <li>Trabajar en el motor de alcohol</li>
    </ul>
  );
}
```

Intenta cambiar los valores de `backgroundColor` y `color`.

Realmente puedes ver el objeto JavaScript dentro de las llaves cuando lo escribes así:

```jsx
<ul style={
  {
    backgroundColor: 'black',
    color: 'pink'
  }
}>
```

**La próxima vez que veas `{{` y `}}` en JSX, ¡sabe que no es más que un objeto dentro de las llaves JSX!**

> [!warning]
>Las propiedades de `style` en línea se escriben en _camelCase_. Por ejemplo, el HTML `<ul style="background-color: black">` se escribiría como `<ul style={{ backgroundColor: 'black' }}>` en tu componente.

## Más diversión con objetos de JavaScript y llaves

Puedes mover varias expresiones a un objeto, y hacer referencia a ellas en tu JSX dentro de llaves:

**App.js**
```jsx
const person = { 👈
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}> 👈
      <h1>Tareas pendientes de {person.name}</h1> 👈
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
      <li>Mejorar el videoteléfono</li>
      <li>Preparar clases de aeronáutica</li>
      <li>Trabajar en el motor de alcohol</li>
      </ul>
    </div>
  );
}
```

En este ejemplo, el objeto JavaScript `person` contiene un string `name` y un objeto `theme`:

```jsx
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};
```

El componente puede usar estos valores de `person` así:

```jsx
<div style={person.theme}>
  <h1>Tareas pendientes de {person.name}</h1>
```

JSX es muy mínimo como lenguaje de plantillas porque te permite organizar datos y lógica usando JavaScript.

## Recapitulación

Ahora ya sabes casi todo sobre JSX:

- Los atributos de JSX dentro de comillas son pasados como strings.
- Las llaves te permiten meter lógica y variables de JavaScript en tu mercado.
- Funcionan dentro del contenido de la etiqueta JSX o inmediatamente después de `=` en los atributos.
- `{{` y `}}` no es una sintaxis especial: es un objeto JavaScript metido dentro de llaves JSX.