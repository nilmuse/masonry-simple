# masonry-simple

Arrange items of different heights in a CSS Grid masonry layout.

[![npm](https://img.shields.io/npm/v/masonry-simple.svg?colorB=brightgreen)](https://www.npmjs.com/package/masonry-simple)
[![NPM Downloads](https://img.shields.io/npm/dm/masonry-simple.svg?style=flat)](https://www.npmjs.com/package/masonry-simple)

[Demo](https://codepen.io/ux-ui/pen/poxGEqX)

## Installation

```bash
npm install masonry-simple
```

## Quick start

Add a grid with items of different heights:

```html
<div class="masonry">
  <article>Short note.</article>
  <article>A longer note with enough text to wrap onto several lines in a narrow column.</article>
  <article>Another short note.</article>
</div>
```

```css
.masonry {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 12px;
}

.masonry > article {
  padding: 24px;
  background: #edf2f7;
}
```

Run this in your browser entry point after the grid markup is available:

```ts
import MasonrySimple from 'masonry-simple';

const masonry = new MasonrySimple({ container: '.masonry' });

masonry.init();
```

The library measures each item's rendered height and sets its grid row span. It schedules updates for container resizing, direct child changes, and image loading.

## API

`new MasonrySimple(options?)` creates a layout controller. Call `init()` to activate it.

### Options

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `container` | `HTMLElement \| string` | `'.masonry'` | Grid container element or selector. |

### Methods

- `init()` finds the container, collects its children, attaches observers and listeners, and schedules a layout. Repeated calls do not duplicate observers or listeners.
- `refresh()` collects the current grid items and schedules one layout pass. Calls before `init()` or after `destroy()` do nothing.
- `destroy()` cancels scheduled animation frame work, disconnects observers, and removes listeners. It clears each tracked item's inline `gridRowEnd` and restores the container's original inline `gridAutoRows`, `contain`, and `alignItems` values.

Call `destroy()` when you remove the grid. You can call `init()` again to reuse the controller.

## Vue 3

Create the controller after the grid mounts and destroy it before unmounting:

```vue
<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref, shallowRef } from 'vue';
import MasonrySimple from 'masonry-simple';

const masonryRef = ref<HTMLElement | null>(null);
const masonry = shallowRef<MasonrySimple | null>(null);

onMounted(() => {
  if (!masonryRef.value) return;
  masonry.value = new MasonrySimple({ container: masonryRef.value });
  masonry.value.init();
});

onBeforeUnmount(() => {
  masonry.value?.destroy();
  masonry.value = null;
});
</script>

<template>
  <div ref="masonryRef" class="masonry">
    <article>Short note.</article>
    <article>A longer note that wraps onto several lines in a narrow column.</article>
    <article>Another short note.</article>
  </div>
</template>

<style scoped>
.masonry {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 12px;
}

.masonry > article {
  padding: 24px;
  background: #edf2f7;
}
</style>
```

## Styling

The container must use CSS Grid. Define its columns and gaps in your stylesheet; the package does not ship CSS.

While active, the library sets these inline container styles:

- `grid-auto-rows: 1px`
- `contain: layout`
- `align-items: start`

It restores their previous inline values on `destroy()`. Item heights come from rendered content, so give images known dimensions when possible to reduce layout changes after loading.

## Layout updates and limitations

- A missing container makes the controller's methods do nothing.
- Initialize in a browser. Importing the module does not initialize a grid, but `init()` requires DOM globals, including `HTMLElement`; calling it in a plain Node.js environment can throw.
- Without `ResizeObserver` or `MutationObserver`, use `refresh()` for changes those observers would otherwise catch.
- A hidden or zero-size container may temporarily receive fallback row spans. Refresh after it becomes visible if no observed change triggers a layout.
- The mutation observer watches direct child additions and removals. For content or size changes that do not trigger an observed container resize or image load, call `refresh()`.

Batch DOM changes before calling `refresh()`, and avoid changing item styles on every animation frame.

## License

MIT
