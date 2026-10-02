# Guía rápida 2 — Reactividad (el corazón de Vue)

> Autocontenida: ejemplos con Composition API (`<script setup>` vía Vite) y notas de equivalencia con Options API.

## 7. `data()` y reactividad en Options API

```html
<div id="app">
  <p>{{ contador }}</p>
  <button @click="contador++">+1</button>
</div>

<script>
  Vue.createApp({
    data() {
      return { contador: 0 } // Vue "envuelve" este objeto para detectar cambios
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** cada clic incrementa el número mostrado en pantalla sin recargar la página.

Internamente, Vue convierte las propiedades de lo que retorna `data()` en reactivas usando `Proxy`. Por eso basta con hacer `this.contador++` y la vista se actualiza sola.

## 8. `ref()` y `reactive()` (Composition API)

```html
<div id="app">
  <p>Contador (ref): {{ contador }}</p>
  <button @click="contador++">+1</button>

  <p>Usuario (reactive): {{ usuario.nombre }}, {{ usuario.edad }} años</p>
  <button @click="usuario.edad++">Cumplir años</button>
</div>

<script>
  const { createApp, ref, reactive } = Vue

  createApp({
    setup() {
      const contador = ref(0) // para valores primitivos (number, string, boolean)
      const usuario = reactive({ nombre: 'Luis', edad: 30 }) // para objetos

      return { contador, usuario }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** ambos botones actualizan su valor en pantalla al hacer clic.

| | `ref()` | `reactive()` |
|---|---|---|
| Úsalo para | primitivos y también objetos | solo objetos/arrays |
| Acceso en JS | `contador.value` (dentro de `setup`) | `usuario.nombre` directo |
| Acceso en template | `contador` (sin `.value`, Vue lo "desenvuelve") | `usuario.nombre` |

**Error común:** olvidar `.value` dentro del `<script>` cuando usas `ref`:
```js
console.log(contador)       // ❌ imprime el objeto RefImpl, no el número
console.log(contador.value) // ✅ imprime el número
```

## 9. `computed()`

```html
<div id="app">
  <input v-model.number="precio" type="number">
  <input v-model.number="cantidad" type="number">
  <p>Total: {{ total }}</p>
</div>

<script>
  const { createApp, ref, computed } = Vue

  createApp({
    setup() {
      const precio = ref(10)
      const cantidad = ref(2)

      // se recalcula SOLO cuando precio o cantidad cambian
      const total = computed(() => precio.value * cantidad.value)

      return { precio, cantidad, total }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** al cambiar precio o cantidad, "Total" se actualiza automáticamente (ej. precio=10, cantidad=2 → Total: 20).

- `computed` **cachea** el resultado: si ninguna de sus dependencias cambia, no vuelve a ejecutar la función.
- Úsalo para "datos derivados" de otros datos (ej. total, filtrado de listas, texto formateado). No lo uses para efectos secundarios (eso es `watch`).

## 10. `watch()` y `watchEffect()`

```html
<div id="app">
  <input v-model="busqueda" placeholder="Buscar...">
  <p>{{ resultado }}</p>
</div>

<script>
  const { createApp, ref, watch } = Vue

  createApp({
    setup() {
      const busqueda = ref('')
      const resultado = ref('Escribe algo para buscar')

      watch(busqueda, (nuevoValor, valorAnterior) => {
        console.log(`Cambió de "${valorAnterior}" a "${nuevoValor}"`)
        resultado.value = nuevoValor
          ? `Buscando: ${nuevoValor}`
          : 'Escribe algo para buscar'
      })

      return { busqueda, resultado }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** al escribir en el input, en consola se ve el cambio de valor anterior a nuevo, y el párrafo muestra "Buscando: &lt;texto&gt;".

- `watch(fuente, callback)`: reacciona a un cambio específico, te da valor nuevo y valor anterior. Ideal para: llamadas a una API cuando cambia un filtro, validaciones, etc.
- `watchEffect(callback)`: ejecuta la función inmediatamente y vuelve a correr cada vez que cambie **cualquier** dato reactivo que use dentro. No te da el valor anterior.

```js
import { watchEffect } from 'vue'

watchEffect(() => {
  console.log(`El valor actual es: ${busqueda.value}`)
})
```

---

### Ejercicio propuesto

Crea un conversor simple de grados Celsius a Fahrenheit:
- Un input numérico para Celsius (`ref`).
- Un `computed` que calcule Fahrenheit: $F = C \times 1.8 + 32$.
- Un `watch` sobre Celsius que muestre en consola un mensaje cuando el valor supere 100.

**Pista:** la fórmula en JS es `celsius.value * 1.8 + 32`.

**Solución:**
```html
<div id="app">
  <input v-model.number="celsius" type="number" placeholder="Grados Celsius">
  <p>{{ celsius }}°C = {{ fahrenheit }}°F</p>
</div>

<script>
  const { createApp, ref, computed, watch } = Vue

  createApp({
    setup() {
      const celsius = ref(0)

      const fahrenheit = computed(() => (celsius.value * 1.8 + 32).toFixed(1))

      watch(celsius, (nuevo) => {
        if (nuevo > 100) {
          console.log('⚠️ Temperatura de ebullición superada')
        }
      })

      return { celsius, fahrenheit }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** al escribir 100 en el input se muestra "100°C = 212.0°F"; al escribir un valor mayor a 100 aparece el mensaje de advertencia en la consola del navegador.
