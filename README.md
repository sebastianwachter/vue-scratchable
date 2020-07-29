# 🦄 vue-scratchable 🏳️‍🌈🧽

A Vue.js wrapper component that turns everything into fun scratch cards. Includes touch support without additional dependencies.

It can also calculate percentage value of the scratchables's cleared area.

## 🎉 Installation

Install it with npm:

```
npm i vue-scratchable
```

Or directly in the browser:

```html
<script src="https://unpkg.com/vue-scratchable@latest/dist/vue-scratchable.umd.min.js"></script>
```

## ✨ Usage

Register the component globally:

```js
import VueScratchable from './components/vue-scratchable.vue';

Vue.component('vue-scratchable', VueScratchable);
```

Or use it locally in your Single File Components:

```js
import VueScratchable from './components/vue-scratchable.vue';

export default {
  /* ... */
  components: {
    VueScratchable,
  },
  /* ... */
};
```

In both cases you are now able to use it in the `<template>` of a component:

```html
<vue-scratchable>
  <h1>Hello Scratchable World</h1>
</vue-scratchable>
```

## ⚙️ Configuration

### 🎰 Slot

| Slot name | Description |
|-----------|-------------|
| default | The content to be scratched free. |

### 💡 Props

| Property | Type | Description |
|----------|------|-------------|
| brushOptions | Object | Configuration object of the "scratcher". See below for possible options. |
| hideOptions | Object | Configuration object of the scratchable layer. See below for possible options. |
| getPercentageCleared | Boolean | Flag to enable the `percentageUpdate` event which emits the amount of cleared paint as percentage. |
| percentageStride | Number | A stride used while calculating the cleared percentage to reduce calculation time. |

#### 🖊️ Brush options

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| size | Number | 20 | Defines the [`lineWidth` attribute.](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/lineWidth) |
| shape | String | round | Defines the [`lineJoin` attribute.](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/lineJoin) |

#### 🙈 Hide options

There are two different types of fill that can be applied to the scratchable area: a solid colour or a [canvas pattern](https://developer.mozilla.org/en-US/docs/Web/API/CanvasPattern). These are differentiated by the `type` property:

##### 🟩 Solid colour

| Property | Type | Description |
|----------|------|-------------|
| type | String | Can be 'color' or 'pattern'. If you want it to be a solid colour you should set it to 'color'. |
| value | String | The colour you want for the fill. Can be any type of colour: hex or rgba. |

Example:

```js
const hide = {
  type: 'color',
  value: '#333',
};
```

##### 🏳️‍🌈 Pattern

| Property | Type | Description |
|----------|------|-------------|
| type | String | Can be 'color' or 'pattern'. If you want it to be a pattern you should set it to 'pattern'. |
| src | String | A link to an image (best case is a repeatable texture). Can be an external Link as well as an imported local asset. |
| repeat | String | Defines whether and in which direction the image should be repeated. Possible values are "repeat", "no-repeat", "repeat-x" and "repeat-y". |

Example:

```js
const hide = {
  type: 'pattern',
  src: 'https://mdn.mozillademos.org/files/222/Canvas_createpattern.png',
  repeat: 'repeat',
};
```

### 🎈 Events

| Event name | Parameter type | Description |
|------------|----------------|-------------|
| percentageUpdate | Number | If the `getPercentageCleared` flag is set the component will emit this event and pass a number calculated from the percentage amount of the cleared area. |

## ✔️ Caveats

1. Img tags with local assets

When using img tags with local imported assets you have to call the `init` method of the component corresponding to the img tag's `@load` event. The `init` method can be accessed through the component's scoped slot.

Example:

```html
<vue-scratchable v-slot="{ init }">
  <img
    :src="require('./img.jpg')"
    @load="init()"
  >
</vue-scratchable>
```

2. Using external images with patterns won't calculate percentage values.

Patterns from external sources can't be used together with the percentage calculation. If done anyways it will resolve in the following CORS error:

```
Error in v-on handler: "SecurityError: Failed to execute 'getImageData' on 'CanvasRenderingContext2D': The canvas has been tainted by cross-origin data."
```

3. Percentage calculation is very taxing on performance

The cleared area percentage calculation has to take every pixel of the canvas into consideration and analyzes whether they are cleared or not. Since this calculation gets called on every draw step this needs a lot of processing power. Because of that this feature is disabled by default and needs to be explicitly activated.

That's also why the `percentageStride` property should be set wisely and adjusted to your needs. It defines how many pixels should be skipped while calculating. This obviously also decreases the percentage's value accuracy.

## 🧾 License
MIT
