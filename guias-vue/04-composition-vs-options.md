# Guía rápida 4 — Composition API vs Options API

> Autocontenida: mismo ejemplo (un carrito simple) resuelto con ambos estilos, para comparar directamente.

## 16. Options API

Estructura clásica: organizas el código **por tipo** (`data`, `methods`, `computed`, `watch`).

```html
<div id="app">
  <p>Productos en carrito: {{ cantidad }}</p>
  <p>Total: ${{ total }}</p>
  <button @click="agregarProducto">Agregar producto ($50)</button>
</div>

<script>
  const { createApp } = Vue

  createApp({
    data() {
      return {
        productos: []
      }
    },
    computed: {
      cantidad() {
        return this.productos.length
      },
      total() {
        return this.productos.reduce((suma, p) => suma + p.precio, 0)
      }
    },
    watch: {
      cantidad(nuevoValor) {
        console.log(`Ahora hay ${nuevoValor} productos`)
      }
    },
    methods: {
      agregarProducto() {
        this.productos.push({ precio: 50 })
      }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** cada clic agrega $50 al total y suma 1 al contador de productos; en consola se ve el mensaje del `watch`.

## 17. Composition API (`setup()` / `<script setup>`)

El mismo ejemplo, pero organizando el código **por funcionalidad** (todo lo del carrito junto, usando funciones de Vue importadas).

```html
<div id="app">
  <p>Productos en carrito: {{ cantidad }}</p>
  <p>Total: ${{ total }}</p>
  <button @click="agregarProducto">Agregar producto ($50)</button>
</div>

<script>
  const { createApp, ref, computed, watch } = Vue

  createApp({
    setup() {
      const productos = ref([])

      const cantidad = computed(() => productos.value.length)
      const total = computed(() =>
        productos.value.reduce((suma, p) => suma + p.precio, 0)
      )

      watch(cantidad, (nuevoValor) => {
        console.log(`Ahora hay ${nuevoValor} productos`)
      })

      function agregarProducto() {
        productos.value.push({ precio: 50 })
      }

      return { cantidad, total, agregarProducto }
    }
  }).mount('#app')
</script>
```
**Resultado esperado:** idéntico al anterior — el comportamiento es el mismo, solo cambia cómo está organizado el código.

**Nota:** en un proyecto real con Vite, esto se simplifica usando `<script setup>`, donde no necesitas la función `setup()` ni el `return`:
```vue
<script setup>
import { ref, computed, watch } from 'vue'

const productos = ref([])
const cantidad = computed(() => productos.value.length)
const total = computed(() => productos.value.reduce((s, p) => s + p.precio, 0))

watch(cantidad, (n) => console.log(`Ahora hay ${n} productos`))

function agregarProducto() {
  productos.value.push({ precio: 50 })
}
</script>
```

### ¿Cuál elegir?

| | Options API | Composition API |
|---|---|---|
| Curva de aprendizaje | Más fácil al inicio | Requiere entender `ref`/`reactive` |
| Proyectos pequeños | Muy legible | Puede sentirse "de más" |
| Proyectos grandes | El código de una misma funcionalidad queda disperso entre `data`, `methods`, `watch`... | Puedes agrupar toda la lógica de una funcionalidad junta, y reutilizarla en composables |
| Recomendación actual de Vue | Soportada, pero Composition API es el estilo moderno recomendado | ✅ Preferido en proyectos nuevos |

## 18. Composables

Un **composable** es una función que empieza con `use` y encapsula lógica reactiva reutilizable (el equivalente en Vue a los "custom hooks" de React).

```js
// archivo: useContador.js
import { ref } from 'vue'

export function useContador(valorInicial = 0) {
  const contador = ref(valorInicial)

  function incrementar() {
    contador.value++
  }

  function reiniciar() {
    contador.value = valorInicial
  }

  return { contador, incrementar, reiniciar }
}
```

Uso dentro de un componente:
```html
<script setup>
import { useContador } from './useContador.js'

const { contador, incrementar, reiniciar } = useContador(10)
</script>

<template>
  <p>{{ contador }}</p>
  <button @click="incrementar">+1</button>
  <button @click="reiniciar">Reiniciar</button>
</template>
```
**Resultado esperado:** el contador inicia en 10, sube con cada clic en "+1", y vuelve a 10 al hacer clic en "Reiniciar". Si usas `useContador()` en **otro** componente, cada uno tendrá su propio estado independiente.

El valor de los composables es que puedes mover "Me gusta" de un post, validación de formularios, lógica de un cronómetro, etc. a una función reutilizable en vez de copiar y pegar código entre componentes.

---

### Ejercicio propuesto

Convierte este componente de Options API a Composition API:

```js
export default {
  data() {
    return { visible: false }
  },
  methods: {
    alternar() {
      this.visible = !this.visible
    }
  }
}
```

**Pista:** necesitas `ref()` para `visible` y una función normal para `alternar`, luego hacer `return` de ambos.

**Solución:**
```js
import { ref } from 'vue'

export default {
  setup() {
    const visible = ref(false) // estado reactivo equivalente a `data()`

    function alternar() {
      visible.value = !visible.value // se usa .value porque estamos en JS, no en el template
    }

    return { visible, alternar } // expone ambos al template
  }
}
```
**Resultado esperado:** el comportamiento es idéntico al original: cada llamada a `alternar()` invierte el valor booleano de `visible`.
