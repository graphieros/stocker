<template>
  <div class="thermometer__container">
    <button
      v-if="showRefreshButton"
      class="mb-11 thermometer__refresh-button"
      @click="
        reinit();
        animate();
      "
    >
      REFRESH
    </button>
    <canvas
      :height="resolution * 1.1"
      :width="resolution * 0.5"
      :class="{ thermometer__canvas: true }"
      ref="thermometerCanvas"
      :style="{ background: backgroundColor, height: `${size}px` }"
    ></canvas>
    <div
      v-show="isTooltip && tooltipHtml"
      class="thermometer__tooltip"
      v-html="tooltipHtml"
      :style="`position: fixed; left: ${mouseX}px; top:${mouseY}px`"
    ></div>
  </div>
</template>

<script>
import Vue from "vue";
export default Vue.extend({
  name: "Thermometer",
  props: {
    animated: {
      type: Boolean,
      default: false,
    },
    colors: {
      type: Array,
      default() {
        return [
          "red",
          "#ff3300",
          "#ff6600",
          "#ff9933",
          "#ffae00",
          "#ffcc00",
          "#ffff00",
          "#ccff33",
          "greenyellow",
          "#5cd65c",
          "#33cc69",
          "#33cc9e",
          "#33ccc9",
          "#33b3cc",
          "#33a6cc",
          "#3399cc",
          "#338acc",
          "#337dcc",
          "#3375cc",
          "#3366cc",
        ];
      },
    },
    dark: {
      type: Boolean,
      default: false,
    },
    darkColor: {
      type: String,
      default: "#18192C",
    },
    hideMeasures: {
      type: Boolean,
      default: false,
    },
    max: {
      type: Number,
      default: 10,
    },
    min: {
      type: Number,
      default: 0,
    },
    range: {
      type: Array,
      default() {
        return [10, 10, 10, 10, 10, 10, 10, 10, 10, 10];
      },
    },
    score: {
      type: Number,
      default: 0,
    },
    speed: {
      type: Number,
      default: 1,
    },
    showRefreshButton: {
      type: Boolean,
      default: false,
    },
    size: {
      type: Number,
      default: 400,
    },
    tooltipHtml: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      acceleration: 1.2,
      initValue: 0,
      isTooltip: false,
      mouseX: 0,
      mouseY: 0,
      resolution: 800,
    };
  },
  computed: {
    backgroundColor() {
      if (this.dark) {
        return this.darkColor;
      }
      return "white";
    },
    canvas() {
      return this.$refs.thermometerCanvas;
    },
    colorRange() {
      const base = this.base100Measures;
      return this.range.map((_measure, i) => {
        return {
          step: base[i],
          color: this.colors[i],
        };
      });
    },
    convertedScore() {
      return this.score;
    },
    ctx() {
      return this.canvas.getContext("2d");
    },
    gap() {
      return this.max - this.min;
    },
    rangeProportion() {
      return [...this.range].map((rect) => {
        return (
          (rect / this.resolution) * this.resolution * (this.resolution / 100)
        );
      });
    },
    tickMap() {
      let gap = (this.max - this.min) * 100;
      const positions = [];
      const x = this.resolution * 0.05;
      for (let i = 0; i <= gap; i += 0.1) {
        const y = (1 - i / this.gap) * this.resolution;
        const value = Number((this.min + i).toFixed(1));
        positions.push({
          x,
          y,
          value,
        });
      }
      return positions;
    },
    textColor() {
      return this.dark ? "white" : this.darkColor;
    },
  },
  methods: {
    getClosestPosition(score) {
      const matchingPosition = this.tickMap.find((el) => {
        return el.value.toFixed(1) === score.toFixed(1);
      });
      return matchingPosition;
    },
    allowTooltip(isVisible) {
      const bounds = this.canvas.getBoundingClientRect();
      if (
        this.mouseX > bounds.y &&
        this.mouseX < bounds.y + bounds.width &&
        this.mouseY > bounds.x &&
        this.mouseY < bounds.x + bounds.height
      ) {
        this.isTooltip = true;
      } else {
        setTimeout(() => {
          this.isTooltip = isVisible;
        }, 50);
      }
    },
    showTooltip(e) {
      e.preventDefault();
      this.mouseX = e.clientX - 100;
      this.mouseY = e.clientY + 30;
    },
    drawRects() {
      const x = this.resolution * 0.2;
      let step = this.resolution * 1.05;
      this.rangeProportion.forEach((proportion, i) => {
        this.ctx.save();
        this.ctx.fillStyle = this.colors[i];
        this.ctx.strokeWidth = 2;
        this.ctx.fillRect(
          x,
          step - proportion,
          this.resolution * 0.11,
          proportion
        );
        this.ctx.restore();
        step -= proportion;
      });
    },
    drawMainTicks() {
      let x = this.resolution * 0.2;
      let y = this.resolution * 0.05;
      let x2 = this.resolution * 0.31;
      this.ctx.lineWidth = 2;
      this.ctx.strokeStyle = "black";
      for (let i = 0; i <= this.resolution; i += this.resolution / this.gap) {
        if (this.gap > 40) {
          if (i % 80 === 0) {
            this.ctx.save();
            this.ctx.beginPath();
            this.ctx.moveTo(x, y + i);
            this.ctx.lineTo(x2, y + i);
            this.ctx.stroke();
            this.ctx.restore();
          }
        } else {
          this.ctx.save();
          this.ctx.beginPath();
          this.ctx.moveTo(x, y + i);
          this.ctx.lineTo(x2, y + i);
          this.ctx.stroke();
          this.ctx.restore();
        }
      }
    },
    drawHalfTicks() {
      let x = this.resolution * 0.2;
      let y = this.resolution * 0.05;
      let x2 = this.resolution * 0.25;
      this.ctx.lineWidth = 1;
      this.ctx.strokeStyle = "black";
      for (
        let i = 0;
        i <= this.resolution;
        i += this.resolution / this.gap / 2
      ) {
        if (this.gap > 40) {
          if (i % 40 === 0) {
            this.ctx.save();
            this.ctx.beginPath();
            this.ctx.moveTo(x, y + i);
            this.ctx.lineTo(x2, y + i);
            this.ctx.stroke();
            this.ctx.restore();
          }
        } else {
          this.ctx.save();
          this.ctx.beginPath();
          this.ctx.moveTo(x, y + i);
          this.ctx.lineTo(x2, y + i);
          this.ctx.stroke();
          this.ctx.restore();
        }
      }
    },
    drawTicks() {
      let x = this.resolution * 0.2;
      let y = this.resolution * 0.05;
      let x2 = this.resolution * 0.225;
      this.ctx.lineWidth = this.resolution * 0.000625;
      this.ctx.strokeStyle = "black";
      for (
        let i = 0;
        i <= this.resolution;
        i += this.resolution / this.gap / 10
      ) {
        if (this.gap > 40) {
          if (Math.round(i) % 20 === 0) {
            this.ctx.save();
            this.ctx.beginPath();
            this.ctx.moveTo(x, y + i);
            this.ctx.lineTo(x2, y + i);
            this.ctx.stroke();
            this.ctx.restore();
          }
        } else {
          this.ctx.save();
          this.ctx.beginPath();
          this.ctx.moveTo(x, y + i);
          this.ctx.lineTo(x2, y + i);
          this.ctx.stroke();
          this.ctx.restore();
        }
      }
    },
    drawPointer(score) {
      let x = this.resolution * 0.175;
      let x2 = this.resolution * 0.2;
      const { y } =
        this.getClosestPosition(score) || this.getClosestPosition(this.min);
      this.ctx.save();
      this.ctx.beginPath();
      this.ctx.strokeStyle = "grey";
      this.ctx.fillStyle = this.textColor;
      this.ctx.strokeWidth = this.resolution * 0.00125;
      this.ctx.moveTo(x2, y + 40);
      this.ctx.lineTo(x, y + 40 - this.resolution * 0.015);
      this.ctx.lineTo(x, y + 40 + this.resolution * 0.015);
      this.ctx.fill();
      this.ctx.stroke();
      this.ctx.restore();
    },
    drawScore(score, source) {
      const coordinates =
        this.getClosestPosition(source) || this.getClosestPosition(this.min);
      const { x, y } = coordinates;
      // let x = this.resolution * 0.05;
      // let ratio = score / this.gap;
      // let y = (1 - ratio) * this.resolution + 52;
      this.ctx.save();
      this.ctx.strokeStyle = "black";
      this.ctx.strokeWidth = this.resolution * 0.0125;
      this.ctx.fillStyle = this.getScoreColor(source);
      this.ctx.font = `900 ${this.resolution * 0.05}px Ubunto Mono, monospace`;
      this.ctx.textAlign = "center";
      this.ctx.fillText(source.toFixed(1), x + 25, y + 54);
      this.ctx.strokeText(source.toFixed(1), x + 25, y + 54);
      this.ctx.restore();
    },
    drawMeasures() {
      let init = this.resolution * 1.0625;
      let x = this.resolution * 0.325;
      let measureSet = [];
      for (let i = this.min; i <= this.max; i += 1) {
        if (this.gap > 40) {
          if (i % 20 === 0) {
            measureSet.push(i);
          } else {
            measureSet.push("");
          }
        } else {
          measureSet.push(i);
        }
      }
      measureSet.forEach((measure) => {
        this.ctx.save();
        this.ctx.fillStyle = this.textColor;
        this.ctx.font = `${this.resolution * 0.035}px Ubunto Mono, monospace`;
        this.ctx.fillText(measure, x, init);
        init -= this.resolution / this.gap;
        this.ctx.restore();
      });
    },
    drawThermometer(score, source) {
      this.drawRects();
      this.drawTicks();
      this.drawHalfTicks();
      this.drawPointer(score);
      this.drawScore(score, source);
      if (!this.hideMeasures) {
        this.drawMeasures();
      }
      this.drawMainTicks();
    },
    animate() {
      this.ctx.save();
      this.ctx.clearRect(0, 0, this.resolution * 0.5, this.resolution * 1.1);
      let tempScore =
        this.initValue > Number(this.convertedScore)
          ? Number(this.convertedScore)
          : this.initValue;

      this.drawThermometer(tempScore, tempScore);

      if (this.initValue < Number(this.convertedScore)) {
        this.initValue += 0.05 * this.acceleration * Number(this.speed);
        this.acceleration += 0.01 * Number(this.speed);
        requestAnimationFrame(this.animate);
      } else {
        cancelAnimationFrame(this.animate);
      }

      this.ctx.restore();
    },
    reinit() {
      this.initValue = 0;
      this.acceleration = 1.2;
      if (this.animated) {
        requestAnimationFrame(this.animate);
      } else {
        this.drawThermometer(this.convertedScore, this.score);
      }
    },
    getScoreColor() {
      if (this.dark) {
        return "white";
      }
      return "black";
    },
  },
  mounted() {
    if (this.animated) {
      requestAnimationFrame(this.animate);
    } else {
      this.drawThermometer(this.convertedScore, this.score);
    }
  },
  updated() {
    if (this.animated) {
      requestAnimationFrame(this.animate);
    } else {
      this.drawThermometer(this.convertedScore, this.score);
    }
  },
});
</script>

<style lang="scss" scoped>
.thermometer {
  font-family: "Ubuntu Mono", monospace;
  position: relative;
  &__canvas {
    background: white;
    border-radius: 36px;
  }
  &__container {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    width: fit-content;
    position: relative;
  }
  &__refresh-button {
    color: grey;
    cursor: pointer;
    border: 1px solid grey;
    padding: 3px 12px;
    border-radius: 6px;
    z-index: 1;
    position: absolute;
    bottom: -26px;
    right: -50px;
  }
  &__tooltip-trap {
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    border-radius: 50%;
  }
  &__tooltip {
    background: white;
    border-radius: 6px;
    box-shadow: 0px 10px 20px -5px rgba(0, 0, 0, 0.247);
    height: fit-content;
    width: 200px;
    color: black;
    padding: 12px;
    z-index: 1000000;
  }
}
</style>
