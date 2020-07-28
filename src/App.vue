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
import VueScratchable from './components/vue-scratchable.vue';
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
