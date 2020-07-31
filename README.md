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
import VueScratchable from 'vue-scratchable';

Vue.component('vue-scratchable', VueScratchable);
```

Or use it locally in your Single File Components:

```js
import VueScratchable from 'vue-scratchable';

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

## 🤔 Complete example as a Single File Component

This code is taken from this project's `App.vue` file to showcase the component's easy way of use.

```html
<template>
  <div id="app">
    <h1>A beautiful parrot got trapped behind some paper.</h1>
    <h2>Scratch them free!</h2>
    <vue-scratchable
      v-slot="{ init }"
      :brushOptions="brush"
      :hideOptions="hide"
      getPercentageCleared
      @percentageUpdate="updatePoints"
    >
      <div class="wrapper">
        <img
          :src="require('./assets/mehmet-turgut-kirkgoz-vnayhPykN5Q-unsplash.jpg')"
          @load="init()"
        >
        <h3>{{ subline }}</h3>
      </div>
    </vue-scratchable>
    <p>You scratched {{ percentage }}% free.</p>
    <pre>Photo by <a href="https://unsplash.com/@tkirkgoz?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Mehmet Turgut Kirkgoz</a> on <a href="https://unsplash.com/t/animals?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></pre>
  </div>
</template>

<script>
import VueScratchable from 'vue-scratchable';
import paperPattern from './assets/natural-paper-texture.jpg';

export default {
  name: 'App',
  components: {
    VueScratchable,
  },
  computed: {
    subline() {
      return this.percentage < 100
        ? `🎉 There is still ${100 - this.percentage}% left for me to be free... 🎉`
        : '💚 Thank you for scratching me free! 💚';
    },
  },
  data() {
    return {
      percentage: 0,
      hide: {
        type: 'pattern',
        src: paperPattern,
        repeat: 'repeat',
      },
      brush: {
        size: 60,
        shape: 'round',
      },
    };
  },
  methods: {
    updatePoints(percentage) {
      this.percentage = percentage;
    },
  },
};
</script>

<style>
body {
  background-color: #333;
  margin: 0;
}

#app {
  font-family: 'Open Sans', sans-serif;
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 1200px;
  margin: 0 auto;
  margin-top: 50px;
}

.vue-scratchable-wrapper {
  background-color: white;
}

h3 {
  color: #2c3e50;
  text-align: center;
}

a {
  color: #2196f3;
}
</style>
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
