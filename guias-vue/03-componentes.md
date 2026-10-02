# Guía rápida 3 — Componentes

> Autocontenida: ejemplos con `Vue.createApp` + `app.component(...)` para que funcionen por CDN, sin necesidad de un proyecto Vite.

## 11. Creación de componentes

```html
<div id="app">
  <tarjeta-saludo></tarjeta-saludo>
  <tarjeta-saludo></tarjeta-saludo>
</div>

<script>
  const { createApp } = Vue

  const app = createApp({})

  app.component('tarjeta-saludo', {
    template: `<div class="tarjeta"><h3>Hola 👋</h3><p>Soy un componente reutilizable</p></div>`
  })

  app.mount('#app')
</script>
```
**Resultado esperado:** se renderizan dos tarjetas idénticas, una por cada `<tarjeta-saludo>` usado en el HTML.

Un componente es una pieza de UI reutilizable con su propio `template` (y opcionalmente `data`, `methods`, etc.), igual que una app completa de Vue pero pensada para insertarse dentro de otra.

## 12. `props`

```html
<div id="app">
  <tarjeta-usuario nombre="Ana" :edad="25"></tarjeta-usuario>
  <tarjeta-usuario nombre="Luis" :edad="30"></tarjeta-usuario>
</div>

<script>
  const { createApp } = Vue
  const app = createApp({})

  app.component('tarjeta-usuario', {
    props: ['nombre', 'edad'],
    template: `<p>{{ nombre }} tiene {{ edad }} años</p>`
  })

  app.mount('#app')
</script>
```
**Resultado esperado:**
```
Ana tiene 25 años
Luis tiene 30 años
```

Las `props` son datos que el **padre** le pasa al **hijo**. El flujo es de una sola dirección (padre → hijo); el hijo **no debe modificar** una prop directamente.

**Con validación de tipos (buena práctica):**
```js
app.component('tarjeta-usuario', {
  props: {
    nombre: { type: String, required: true },
    edad: { type: Number, default: 18 }
  },
  template: `<p>{{ nombre }} tiene {{ edad }} años</p>`
})
```

## 13. `emits` (eventos personalizados)

```html
<div id="app">
  <p>Likes: {{ likes }}</p>
  <boton-like @dar-like="likes++"></boton-like>
</div>

<script>
  const { createApp } = Vue
  const app = createApp({
    data() {
      return { likes: 0 }
    }
  })

  app.component('boton-like', {
    emits: ['dar-like'],
    template: `<button @click="$emit('dar-like')">👍 Me gusta</button>`
  })

  app.mount('#app')
</script>
```
**Resultado esperado:** cada clic en el botón del componente hijo incrementa el contador "Likes" que vive en el padre.

El hijo **avisa** al padre con `$emit('nombre-evento', datosOpcionales)`; el padre escucha ese evento con `@nombre-evento="..."`, igual que escucharía un `click` nativo.

## 14. `slots`

```html
<div id="app">
  <tarjeta>
    <h3>Título personalizado</h3>
    <p>Este contenido lo define quien usa el componente.</p>
  </tarjeta>
</div>

<script>
  const { createApp } = Vue
  const app = createApp({})

  app.component('tarjeta', {
    template: `
      <div style="border:1px solid #ccc; padding:10px;">
        <slot></slot>
      </div>
    `
  })

  app.mount('#app')
</script>
```
**Resultado esperado:** el `<h3>` y `<p>` escritos dentro de `<tarjeta>` aparecen dentro del `<div>` con borde, en el lugar donde está `<slot></slot>`.

Un `slot` es un "hueco" en el template del componente donde se inserta el contenido que le pasas desde afuera, útil para componentes genéricos (modales, tarjetas, layouts).

## 15. Ciclo de vida de un componente

```html
<div id="app">
  <button @click="mostrar = !mostrar">Alternar componente</button>
  <reloj v-if="mostrar"></reloj>
</div>

<script>
  const { createApp, onMounted, onUnmounted, ref } = Vue
  const app = createApp({
    data() {
      return { mostrar: true }
    }
  })

  app.component('reloj', {
    template: `<p>Hora: {{ hora }}</p>`,
    setup() {
      const hora = ref(new Date().toLocaleTimeString())
      let intervalo

      onMounted(() => {
        console.log('✅ Componente montado')
        intervalo = setInterval(() => {
          hora.value = new Date().toLocaleTimeString()
        }, 1000)
      })

      onUnmounted(() => {
        console.log('🧹 Componente desmontado, limpiando intervalo')
        clearInterval(intervalo)
      })

      return { hora }
    }
  })

  app.mount('#app')
</script>
```
**Resultado esperado:** al mostrar el componente, el reloj actualiza la hora cada segundo y en consola aparece "Componente montado"; al ocultarlo con el botón, aparece "Componente desmontado" y el reloj deja de actualizarse (sin fugas de memoria).

| Hook | Cuándo se ejecuta |
|---|---|
| `onMounted` | Cuando el componente ya se insertó en el DOM |
| `onUpdated` | Cada vez que el componente se vuelve a renderizar |
| `onUnmounted` | Cuando el componente se elimina del DOM |

---

### Ejercicio propuesto

Crea un componente `contador-saludo` que:
- Reciba una `prop` llamada `nombre`.
- Tenga su propio estado interno `veces` (cuántas veces se le dio clic).
- Emita un evento `limite-alcanzado` cuando `veces` llegue a 3, y el padre muestre una alerta.

**Pista:** usa `data()` para `veces` dentro del componente, y un `watch` o un `if` dentro del método del clic para emitir el evento.

**Solución:**
```html
<div id="app">
  <contador-saludo nombre="Ana" @limite-alcanzado="alertarLimite"></contador-saludo>
</div>

<script>
  const { createApp } = Vue
  const app = createApp({
    methods: {
      alertarLimite() {
        alert('¡Llegaste al límite de clics!')
      }
    }
  })

  app.component('contador-saludo', {
    props: ['nombre'],
    emits: ['limite-alcanzado'],
    data() {
      return { veces: 0 }
    },
    template: `
      <div>
        <p>Hola {{ nombre }}, clics: {{ veces }}</p>
        <button @click="incrementar">Saludar</button>
      </div>
    `,
    methods: {
      incrementar() {
        this.veces++
        if (this.veces === 3) {
          this.$emit('limite-alcanzado')
        }
      }
    }
  })

  app.mount('#app')
</script>
```
**Resultado esperado:** el contador de clics sube con cada clic; al llegar al tercer clic, se dispara una alerta del navegador generada en el componente padre.
