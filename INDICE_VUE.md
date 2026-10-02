# Índice de aprendizaje de Vue.js

Guía de temas, ordenados de básico a avanzado, para dominar Vue.js desde cero.

## 1. Fundamentos de Vue

1. **¿Qué es Vue y por qué usarlo?** — Entender qué problema resuelve frente a HTML/JS plano (reactividad, componentes, DOM virtual).
2. **Instancia de la aplicación (`createApp`)** — Cómo se monta una app de Vue en el DOM con `mount()`.
3. **Interpolación de texto (`{{ }}`)** — Mostrar datos de JavaScript dentro del HTML.
4. **Directivas básicas** — `v-bind`, `v-if`/`v-else`/`v-show`, `v-for`, `v-on`: cómo Vue manipula el DOM de forma declarativa.
5. **Manejo de eventos** — Escuchar clics, inputs y otros eventos con `v-on` (`@click`, `@input`, etc.).
6. **Formularios y `v-model`** — Enlace bidireccional de datos entre inputs y el estado de la app.

## 2. Reactividad (el corazón de Vue)

7. **`data()` y reactividad en Options API** — Cómo Vue detecta cambios y actualiza la vista automáticamente.
8. **`ref()` y `reactive()` (Composition API)** — Las dos formas de crear estado reactivo y cuándo usar cada una.
9. **`computed()`** — Propiedades derivadas que se recalculan solo cuando cambian sus dependencias.
10. **`watch()` y `watchEffect()`** — Reaccionar a cambios de estado para ejecutar efectos secundarios (llamadas a APIs, logs, etc.).

## 3. Componentes

11. **Creación de componentes** — Dividir la interfaz en piezas reutilizables.
12. **`props`** — Pasar datos de un componente padre a un hijo.
13. **`emits` (eventos personalizados)** — Comunicar datos de un componente hijo hacia el padre.
14. **`slots`** — Insertar contenido dinámico dentro de un componente desde fuera.
15. **Ciclo de vida de un componente** — `onMounted`, `onUpdated`, `onUnmounted`, etc., y cuándo usarlos.

## 4. Composition API vs Options API

16. **Options API** — Estructura clásica con `data`, `methods`, `computed`, `watch`.
17. **Composition API (`setup()` / `<script setup>`)** — Organización del código por lógica en vez de por tipo, ideal para proyectos grandes.
18. **Composables** — Crear funciones reutilizables (`useX()`) que encapsulan lógica reactiva compartida entre componentes.

## 5. Herramientas del ecosistema

19. **Single File Components (`.vue`)** — Estructura `<template>`, `<script>`, `<style>` en un mismo archivo.
20. **Vite + proyecto Vue** — Cómo crear y ejecutar un proyecto Vue profesional (`npm create vite`).
21. **Vue Router** — Navegación entre "páginas" (vistas) dentro de una Single Page Application.
22. **Pinia** — Manejo de estado global compartido entre varios componentes (sustituto moderno de Vuex).

## 6. Buenas prácticas y estructura de proyecto

23. **Organización de carpetas** — `components/`, `views/`, `router/`, `stores/`, convenciones de nombres.
24. **Props y validación de tipos** — Definir props con tipos y valores por defecto para evitar errores.
25. **Estilos con `scoped`** — Encapsular CSS por componente sin que afecte al resto de la app.
26. **Rendimiento y buenas prácticas de reactividad** — Evitar recálculos innecesarios, usar `key` correctamente en `v-for`, no mutar props directamente.

---

> 💡 **Cómo usar este índice:** avanza en orden, no te saltes el bloque de Reactividad (sección 2), ya que es la base de todo lo demás en Vue. Pide ejercicios de cada tema antes de pasar al siguiente.
