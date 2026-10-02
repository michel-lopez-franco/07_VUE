# Guía rápida 1 — Fundamentos de Vue

> Autocontenida: puedes copiar cualquier bloque de código en un archivo `.html` y abrirlo en el navegador (usa Vue por CDN).

## 1. ¿Qué es Vue y por qué usarlo?

Vue es un framework de JavaScript para construir interfaces reactivas: cuando cambian tus datos, el HTML se actualiza solo, sin que tú tengas que buscar elementos con `document.querySelector` y modificarlos a mano.

**Sin Vue (JS plano):**
```html
<p id="msg">Hola</p>
<button onclick="document.getElementById('msg').innerText = 'Cambiado'">Cambiar</button>
```
Tienes que decirle al DOM *qué* cambiar paso a paso.

**Con Vue:**
```html
<p>{{ msg }}</p>
<button @click="msg = 'Cambiado'">Cambiar</button>
```
Tú solo cambias el dato (`msg`) y Vue decide qué parte del DOM actualizar.

## 2. Instancia de la aplicación (`createApp`)

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body>
  <div id="app">
    <h1>{{ titulo }}</h1>
  </div>

  <script>
    const { createApp } = Vue

    createApp({
      data() {
        return {
          titulo: 'Mi primera app Vue'
        }
      }
    }).mount('#app')
  </script>
</body>
</html>
```
**Resultado esperado:** en el navegador se ve el texto "Mi primera app Vue" dentro del `<h1>`.

- `createApp({...})` crea la aplicación con su configuración (datos, métodos, etc.).
- `.mount('#app')` conecta esa configuración con el elemento del DOM que tenga `id="app"`.
- Todo lo que esté **fuera** de `#app` no es controlado por Vue.

## 3. Interpolación de texto (`{{ }}`)

```html
<div id="app">
  <p>Usuario: {{ nombre }}</p>
  <p>Edad en 5 años: {{ edad + 5 }}</p>
  <p>{{ nombre.toUpperCase() }}</p>
</div>

<script>
  Vue.createApp({
    data() {
      return { nombre: 'Ana', edad: 20 }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:**
```
Usuario: Ana
Edad en 5 años: 25
ANA
```
Dentro de `{{ }}` puedes escribir expresiones JavaScript simples (no sentencias como `if` completos).

## 4. Directivas básicas

| Directiva | Para qué sirve | Ejemplo |
|---|---|---|
| `v-bind` (o `:`) | Enlazar un atributo HTML a una variable | `:src="imagenUrl"` |
| `v-if` / `v-else` | Mostrar u ocultar quitando/poniendo del DOM | `<p v-if="activo">Visible</p>` |
| `v-show` | Mostrar u ocultar con CSS (`display:none`) | `<p v-show="activo">Visible</p>` |
| `v-for` | Repetir un elemento por cada item de una lista | `<li v-for="f in frutas">{{ f }}</li>` |
| `v-on` (o `@`) | Escuchar eventos | `@click="hacerAlgo"` |

```html
<div id="app">
  <img :src="imagen" :alt="descripcion" width="100">

  <p v-if="conectado">Estás conectado ✅</p>
  <p v-else>Sin conexión ❌</p>

  <ul>
    <li v-for="fruta in frutas" :key="fruta">{{ fruta }}</li>
  </ul>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        imagen: 'https://vuejs.org/images/logo.png',
        descripcion: 'Logo de Vue',
        conectado: true,
        frutas: ['Manzana', 'Pera', 'Uva']
      }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** se ve la imagen del logo, el texto "Estás conectado ✅" y una lista con 3 frutas.

> `:key` en `v-for` es obligatorio: ayuda a Vue a identificar cada elemento de la lista para actualizar solo lo que cambió.

## 5. Manejo de eventos

```html
<div id="app">
  <button @click="saludar">Saludar</button>
  <button @click="contador++">Sumé: {{ contador }}</button>
  <input @keyup.enter="mensaje = 'Enviaste con Enter'">
  <p>{{ mensaje }}</p>
</div>

<script>
  Vue.createApp({
    data() {
      return { contador: 0, mensaje: '' }
    },
    methods: {
      saludar() {
        alert('¡Hola!')
      }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** al hacer clic en "Saludar" aparece una alerta; cada clic en el segundo botón suma 1; al presionar Enter en el input cambia el mensaje.

- `@click="contador++"` funciona porque dentro de `@evento` también puedes escribir expresiones cortas.
- `.enter` es un **modificador de evento**: filtra el evento para que solo dispare con la tecla Enter. Otros comunes: `.prevent`, `.stop`, `.once`.

## 6. Formularios y `v-model`

```html
<div id="app">
  <input v-model="nombre" placeholder="Escribe tu nombre">
  <p>Hola, {{ nombre || '...' }}</p>

  <input type="checkbox" v-model="aceptaTerminos">
  <label>Acepto términos</label>
  <p>{{ aceptaTerminos ? 'Aceptado' : 'No aceptado' }}</p>
</div>

<script>
  Vue.createApp({
    data() {
      return { nombre: '', aceptaTerminos: false }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** al escribir en el input, el texto "Hola, ..." cambia en tiempo real; al marcar el checkbox, el texto pasa de "No aceptado" a "Aceptado".

`v-model` es azúcar sintáctica que combina `:value` + `@input` (enlace **bidireccional**): el dato actualiza la vista y la vista actualiza el dato.

---

### Ejercicio propuesto

Crea una mini "lista de tareas" (sin borrar ni editar, solo agregar) con:
- Un input de texto + `v-model`.
- Un botón "Agregar" que meta el texto a un array.
- Un `v-for` que muestre la lista.

**Pista:** usa `tareas.push(nuevaTarea)` dentro de un método, y limpia el input después.

**Solución:**
```html
<div id="app">
  <input v-model="nuevaTarea" placeholder="Nueva tarea" @keyup.enter="agregarTarea">
  <button @click="agregarTarea">Agregar</button>

  <ul>
    <li v-for="(tarea, i) in tareas" :key="i">{{ tarea }}</li>
  </ul>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        nuevaTarea: '',
        tareas: []
      }
    },
    methods: {
      agregarTarea() {
        if (this.nuevaTarea.trim() === '') return // evita tareas vacías
        this.tareas.push(this.nuevaTarea)
        this.nuevaTarea = '' // limpia el input
      }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** cada vez que escribes algo y presionas Enter (o clic en "Agregar"), aparece un nuevo `<li>` en la lista y el input queda vacío.
