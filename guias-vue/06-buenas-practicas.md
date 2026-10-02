# Guía rápida 6 — Buenas prácticas y estructura de proyecto

> Autocontenida: enfocada en errores comunes de principiante a intermedio y cómo evitarlos.

## 23. Organización de carpetas

Estructura recomendada para un proyecto Vite + Vue mediano:

```
src/
├── main.js
├── App.vue
├── router/
│   └── index.js
├── stores/
│   └── carrito.js
├── views/            # componentes que representan "páginas" completas
│   ├── Inicio.vue
│   └── Acerca.vue
├── components/       # componentes pequeños y reutilizables
│   ├── BotonPrimario.vue
│   └── TarjetaProducto.vue
└── composables/
    └── useContador.js
```

**Convenciones:**
- Nombres de componentes en **PascalCase** (`TarjetaProducto.vue`), y al usarlos en el template también en PascalCase o kebab-case (`<TarjetaProducto />` o `<tarjeta-producto />`).
- `views/` para componentes asociados a una ruta de Vue Router; `components/` para piezas reutilizables sin ruta propia.
- Un componente, un archivo: evita archivos `.vue` de más de ~200-300 líneas; si crece mucho, probablemente debas dividirlo.

## 24. Props y validación de tipos

**❌ Mal (sin validación):**
```js
props: ['precio', 'disponible']
```
Si alguien pasa `precio="diez"` (string) en vez de un número, Vue no avisa nada y puedes tener bugs silenciosos.

**✅ Bien (con validación):**
```js
props: {
  precio: {
    type: Number,
    required: true,
    validator: (valor) => valor >= 0 // valida reglas de negocio
  },
  disponible: {
    type: Boolean,
    default: true
  }
}
```
**Resultado esperado:** si en desarrollo pasas un tipo incorrecto (ej. `precio="diez"`), Vue muestra una advertencia clara en la consola del navegador, ayudándote a detectar el error antes de que llegue a producción.

**Regla de oro:** nunca mutes una prop directamente dentro del hijo:
```js
// ❌ Mal
props: ['contador'],
methods: {
  incrementar() {
    this.contador++ // Vue lanza advertencia: "Avoid mutating a prop directly"
  }
}

// ✅ Bien: usa un emit para pedirle al padre que cambie el dato
props: ['contador'],
emits: ['incrementar'],
methods: {
  incrementar() {
    this.$emit('incrementar')
  }
}
```

## 25. Estilos con `scoped`

```vue
<template>
  <p class="titulo">Hola</p>
</template>

<style scoped>
.titulo {
  color: blue;
}
</style>
```
**Resultado esperado:** Vue agrega automáticamente un atributo único (ej. `data-v-7a1b2c3`) a los elementos de este componente y a las reglas CSS, de modo que `.titulo { color: blue; }` **solo** aplica a este componente, aunque otro componente también tenga una clase `.titulo`.

Sin `scoped`, los estilos son **globales** y pueden chocar entre componentes sin que te des cuenta.

**Para estilos realmente globales (ej. reset de CSS), usa un archivo aparte sin `scoped`:**
```vue
<style>
/* styles/global.css importado en main.js, sin scoped */
body { margin: 0; font-family: sans-serif; }
</style>
```

## 26. Rendimiento y buenas prácticas de reactividad

### a) `:key` correcto en `v-for`

**❌ Mal (usar el índice cuando la lista puede reordenarse/eliminar elementos):**
```html
<li v-for="(tarea, index) in tareas" :key="index">{{ tarea.texto }}</li>
```
Si eliminas el primer elemento, Vue reutiliza mal los nodos del DOM y puede mostrar datos desincronizados (ej. un input con el valor "pegado" al elemento equivocado).

**✅ Bien (usar un id único y estable):**
```html
<li v-for="tarea in tareas" :key="tarea.id">{{ tarea.texto }}</li>
```

### b) No mutar props directamente

Ya visto en el tema 24: usa `emits` en vez de modificar `props` desde el hijo.

### c) Evitar cálculos pesados directamente en el template

**❌ Mal:**
```html
<p>{{ productos.filter(p => p.activo).length }}</p>
```
Esto se reevalúa en **cada render**, aunque nada relacionado haya cambiado.

**✅ Bien:**
```js
const productosActivos = computed(() =>
  productos.value.filter(p => p.activo)
)
```
```html
<p>{{ productosActivos.length }}</p>
```
`computed` cachea el resultado y solo se recalcula si `productos` cambia (tema 9).

### d) No sobre-usar `watch`

Si puedes resolver algo con `computed` (un valor derivado de otro), prefiere `computed` sobre `watch`. Usa `watch` solo para **efectos secundarios** (llamadas HTTP, `console.log`, guardar en `localStorage`, etc.), no para calcular datos.

---

### Ejercicio propuesto

Te dan este componente con 3 problemas de buenas prácticas. Encuéntralos y corrígelos.

```vue
<template>
  <li v-for="(item, i) in items" :key="i">
    {{ item.nombre }} - {{ item.precio }}
  </li>
  <p>Total: {{ items.reduce((s, i) => s + i.precio, 0) }}</p>
  <button @click="items[0].precio = 0">Poner primero gratis (prop)</button>
</template>

<script setup>
const props = defineProps(['items'])
</script>
```

**Pista:** son los temas 24 (mutación de prop), 26a (`:key` con índice) y 26c (cálculo pesado en template).

**Solución:**
```vue
<template>
  <li v-for="item in items" :key="item.id">
    {{ item.nombre }} - {{ item.precio }}
  </li>
  <p>Total: {{ total }}</p>
  <button @click="emit('poner-gratis', items[0]?.id)">Poner primero gratis</button>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  items: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['poner-gratis'])

// computed en vez de calcular en cada render dentro del template
const total = computed(() =>
  props.items.reduce((suma, item) => suma + item.precio, 0)
)
</script>
```
**Resultado esperado:**
1. `:key="item.id"` usa un identificador estable en vez del índice.
2. El total se calcula con `computed` (se cachea, no se recalcula en cada render innecesariamente).
3. En vez de mutar `items[0].precio` directamente (prohibido en props), el componente emite un evento `poner-gratis` para que el **padre**, dueño real del dato, decida cómo actualizarlo.
