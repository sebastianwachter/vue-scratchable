<template>
  <div class="vue-scratchable-wrapper">
    <slot :init="init"></slot>
    <canvas
      @mousedown="mouseDown"
      @mousemove="mouseMove"
      @touchstart="touchDown"
      @touchmove="touchMove"
    />
  </div>
</template>

<script>
import debounce from 'lodash/debounce';

export default {
  name: 'VueScratchable',
  props: {
    brushOptions: {
      type: Object,
      default: () => ({
        size: 20,
        shape: 'round',
      }),
    },
    hideOptions: {
      type: Object,
      default: () => ({
        type: 'color',
        value: '#dadada',
      }),
    },
    getPercentageCleared: {
      type: Boolean,
      default: false,
    },
    percentageStride: {
      type: Number,
      default: 150,
    },
  },
  data() {
    return {
      canvas: null,
      context: null,
      isPressed: false,
      offset: {
        top: 0,
        left: 0,
      },
      position: {
        currX: 0,
        currY: 0,
        lastX: 0,
        lastY: 0,
      },
      observer: null,
      initFlag: false,
    };
  },
  mounted() {
    this.canvas = this.$el.querySelector('canvas');
    const debounceInit = debounce(() => this.init(), 200);

    this.observer = new MutationObserver((mutations) => {
      mutations.forEach(({ attributeName }) => {
        if (this.initFlag || !attributeName) return;
        debounceInit();
      });
    });
    this.observer.observe(this.$el, {
      childList: true,
      attributes: true,
      characterData: true,
      subtree: true,
    });

    window.addEventListener('mouseup', this.mouseUp, false);
    window.addEventListener('touchend', this.touchUp, false);
    window.addEventListener('touchcancel', this.touchUp, false);

    this.init();

    window.addEventListener('resize', debounce(this.init, 500));
  },
  destroyed() {
    window.removeEventListener('mouseup', this.mouseUp);
    window.removeEventListener('touchend', this.touchUp);
    window.removeEventListener('touchcancel', this.touchUp);

    window.removeEventListener('resize', debounce(this.init, 500));

    this.observer.disconnect();
  },
  methods: {
    init() {
      this.initFlag = true;
      this.setCanvasSizeAndContext();
      this.$nextTick(() => this.fillArea());
    },

    setCanvasSizeAndContext() {
      this.$nextTick(() => {
        const { width, height } = this.$el.getBoundingClientRect();
        this.canvas.width = Math.ceil(width);
        this.canvas.height = Math.ceil(height);
        this.context = this.canvas.getContext('2d');
      });
    },

    setOffsets() {
      const { top, left } = this.canvas.getBoundingClientRect();
      this.offset.top = top + document.body.scrollTop;
      this.offset.left = left + document.body.scrollLeft;
    },

    async fillArea() {
      const { width, height } = this.context.canvas;
      await this.setFillStyle()
        .catch((err) => {
          // eslint-disable-next-line no-console
          console.error(` Failed to load image!
            Error: ${err.name}
            Message: ${err.message}
          `);
        });
      this.context.fillRect(0, 0, width, height);
      this.initFlag = false;
    },

    setFillStyle() {
      const {
        type,
        value = '',
        src = '',
        repeat = '',
      } = this.hideOptions;
      this.context.globalCompositeOperation = 'source-over';

      // eslint-disable-next-line no-return-assign
      if (type === 'color') return new Promise((resolve) => resolve(this.context.fillStyle = value));

      return new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => {
          this.context.fillStyle = this.context.createPattern(img, repeat);
          resolve(img);
        };
        img.onerror = (err) => reject(err);
        img.src = src;
      });
    },

    clearArea() {
      const { width, height } = this.context.canvas;
      this.context.globalCompositeOperation = 'destination-out';
      this.context.fillRect(0, 0, width, height);
    },

    draw() {
      const {
        currX,
        currY,
        lastX,
        lastY,
      } = this.position;
      this.context.beginPath();
      this.context.globalCompositeOperation = 'destination-out';
      this.context.lineWidth = this.brushOptions.size;
      this.context.lineJoin = this.brushOptions.shape;
      this.context.moveTo(lastX, lastY);
      if (this.isPressed) {
        this.context.lineTo(currX, currY);
      } else {
        this.context.lineTo(currX + 0.01, currY + 0.01);
      }
      this.context.closePath();
      this.context.stroke();
      this.lastPositionHelper(currX, currY);
      this.calculateClearedArea();
    },

    lastPositionHelper(x, y) {
      this.position.lastX = x;
      this.position.lastY = y;
    },
    currentPositionHelper(x, y) {
      this.position.currX = x - this.offset.left;
      this.position.currY = y - this.offset.top;
    },

    mouseDown({ clientX, clientY }) {
      this.setOffsets();
      this.currentPositionHelper(clientX, clientY);
      this.lastPositionHelper(this.position.currX, this.position.currY);
      this.draw();
      this.isPressed = true;
    },
    mouseMove({ clientX, clientY }) {
      if (!this.isPressed) return;
      this.currentPositionHelper(clientX, clientY);
      this.draw();
    },
    mouseUp() {
      this.isPressed = false;
    },

    touchDown({ targetTouches: [{ clientX, clientY }] }) {
      this.setOffsets();
      this.currentPositionHelper(clientX, clientY);
      this.lastPositionHelper(this.position.currX, this.position.currY);
      this.draw();
      this.isPressed = true;
    },
    touchMove(event) {
      event.preventDefault();
      const { targetTouches: [{ clientX, clientY }] } = event;
      this.currentPositionHelper(clientX, clientY);
      this.draw();
    },
    touchUp() {
      this.isPressed = false;
    },

    calculateClearedArea() {
      if (!this.getPercentageCleared) return;
      const { width, height } = this.context.canvas;
      const { data, data: { length } } = this.context.getImageData(0, 0, width, height);
      const total = length / this.percentageStride;
      let clearedCount = 0;
      for (let i = clearedCount; i < length; i += this.percentageStride) {
        if (parseInt(data[i], 10) === 0) clearedCount += 1;
      }
      this.$emit('percentage-update', Math.round((clearedCount / total) * 100));
    },
  },
};
</script>

<style scoped>
.vue-scratchable-wrapper {
  position: relative;
}

.vue-scratchable-wrapper > * {
  user-select: none;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
