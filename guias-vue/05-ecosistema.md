# Guía rápida 5 — Herramientas del ecosistema

> Autocontenida: aquí sí se requiere Node.js instalado, porque Router y Pinia necesitan un proyecto con Vite (no funcionan bien solo con CDN).

## 19. Single File Components (`.vue`)

Un **SFC** (Single File Component) junta HTML, JS y CSS de un componente en un solo archivo `.vue`:

```vue
<!-- Saludo.vue -->
<template>
  <p :class="{ resaltado: esImportante }">{{ mensaje }}</p>
</template>

<script setup>
import { ref } from 'vue'

const mensaje = ref('Hola desde un Single File Component')
const esImportante = ref(true)
</script>

<style scoped>
.resaltado {
  color: red;
  font-weight: bold;
}
</style>
```
**Resultado esperado:** se muestra el texto en rojo y negrita, y ese estilo `.resaltado` **no afecta** a otros componentes de la app (gracias a `scoped`, tema 25).

Esto solo funciona con un **build tool** como Vite, que compila el `.vue` a JavaScript normal antes de servirlo al navegador.

## 20. Vite + proyecto Vue

Pasos para crear un proyecto real (no por CDN):

```bash
npm create vite@latest mi-proyecto-vue -- --template vue
cd mi-proyecto-vue
npm install
npm run dev
```
**Resultado esperado:** el último comando levanta un servidor local (normalmente `http://localhost:5173`) donde ves la app de ejemplo que trae Vite por defecto.

Estructura típica generada:
```
mi-proyecto-vue/
├── index.html
├── package.json
├── vite.config.js
└── src/
    ├── main.js
    ├── App.vue
    └── components/
        └── HelloWorld.vue
```

`src/main.js` es el punto de entrada:
```js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

> 💡 Si quieres, puedo crear este proyecto por ti usando el terminal de VS Code.

## 21. Vue Router

Permite tener varias "páginas" (vistas) sin recargar el navegador (Single Page Application).

```bash
npm install vue-router@4
```

```js
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Inicio from '../views/Inicio.vue'
import Acerca from '../views/Acerca.vue'

const routes = [
  { path: '/', component: Inicio },
  { path: '/acerca', component: Acerca }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

```js
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```

```vue
<!-- App.vue -->
<template>
  <nav>
    <router-link to="/">Inicio</router-link>
    <router-link to="/acerca">Acerca de</router-link>
  </nav>

  <router-view></router-view>
</template>
```
**Resultado esperado:** al hacer clic en "Inicio" o "Acerca de" cambia el contenido de `<router-view>` sin recargar la página, y la URL del navegador cambia (`/` o `/acerca`).

- `<router-link>` genera un `<a>` que navega sin recargar.
- `<router-view>` es el "hueco" donde se renderiza el componente de la ruta activa.

## 22. Pinia

Maneja estado **global**, compartido entre componentes que no tienen relación directa padre-hijo (ej. el usuario logueado, un carrito de compras visible en toda la app).

```bash
npm install pinia
```

```js
// src/main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

createApp(App).use(createPinia()).mount('#app')
```

```js
// src/stores/carrito.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCarritoStore = defineStore('carrito', () => {
  const productos = ref([])

  const total = computed(() =>
    productos.value.reduce((suma, p) => suma + p.precio, 0)
  )

  function agregar(producto) {
    productos.value.push(producto)
  }

  return { productos, total, agregar }
})
```

Uso en **cualquier** componente de la app:
```vue
<script setup>
import { useCarritoStore } from '../stores/carrito'

const carrito = useCarritoStore()
</script>

<template>
  <p>Total: ${{ carrito.total }}</p>
  <button @click="carrito.agregar({ precio: 99 })">Agregar producto</button>
</template>
```
**Resultado esperado:** si usas `useCarritoStore()` en dos componentes distintos (ej. un `<Navbar>` y una `<PaginaProductos>`), ambos ven el **mismo** total actualizado, porque comparten la misma "fuente de verdad" (el store), a diferencia de `props`/`emits` que solo comunican padre-hijo.

---

### Ejercicio propuesto

(Requiere un proyecto Vite ya creado.) Crea un store de Pinia llamado `useTemaStore` que:
- Guarde un estado `modoOscuro` (booleano, inicia en `false`).
- Tenga una acción `alternarTema()` que invierta el valor.
- Úsalo en dos componentes distintos y verifica que el cambio se refleje en ambos.

**Pista:** la estructura es igual al ejemplo del carrito, pero con un solo booleano en vez de un array.

**Solución:**
```js
// src/stores/tema.js
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useTemaStore = defineStore('tema', () => {
  const modoOscuro = ref(false)

  function alternarTema() {
    modoOscuro.value = !modoOscuro.value
  }

  return { modoOscuro, alternarTema }
})
```

```vue
<!-- ComponenteA.vue -->
<script setup>
import { useTemaStore } from '../stores/tema'
const tema = useTemaStore()
</script>

<template>
  <button @click="tema.alternarTema">Alternar tema</button>
</template>
```

```vue
<!-- ComponenteB.vue (en otra parte de la app) -->
<script setup>
import { useTemaStore } from '../stores/tema'
const tema = useTemaStore()
</script>

<template>
  <p>Modo oscuro activo: {{ tema.modoOscuro }}</p>
</template>
```
**Resultado esperado:** al hacer clic en el botón de `ComponenteA`, el texto en `ComponenteB` cambia de `false` a `true` (y viceversa), sin que ambos componentes tengan relación directa entre sí.
