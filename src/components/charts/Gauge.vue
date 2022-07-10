<template>
  <div class="gauge__container">
    <div>
      <v-btn
        v-if="showRefreshButton"
        class="mt-n3"
        absolute
        bottom
        small
        outlined
        fab
        :color="dark ? 'grey' : darkColor"
        @click="
          reinit();
          animate();
        "
        ><v-icon>mdi-refresh</v-icon></v-btn
      >

      <div
        v-if="tooltipHtml"
        @mouseover="isTooltip = true"
        @mouseleave="isTooltip = false"
        class="gauge__tooltip-trap"
      ></div>

      <canvas
        height="400"
        width="400"
        class="gauge__canvas"
        ref="customGaugeCanvas"
        :style="{ background: colorTheme.recto, height: `${size}px` }"
      ></canvas>
      <div
        v-show="isTooltip && tooltipHtml"
        @mouseover="isTooltip = true"
        @mouseleave="isTooltip = false"
        class="gauge__tooltip"
        v-html="tooltipHtml"
        :style="`border: 1px solid ${getScoreColor(score)}`"
      ></div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
export default Vue.extend({
  name: "FlexGauge",
  components: {},
  props: {
    animated: {
      type: Boolean,
      default: false,
    },
    animationSpeed: {
      type: Number,
      default: 0,
    },
    acceleration: {
      type: Number,
      default: 0.3,
    },
    base5: {
      type: Boolean,
      default: false,
    },
    base10: {
      type: Boolean,
      default: false,
    },
    base100: {
      type: Boolean,
      default: false,
    },
    colors: {
      type: Array,
      default() {
        return [
          "#fc0303",
          "#ff3300",
          "#ff6600",
          "#ff9933",
          "#ffae00",
          "#ffcc00",
          "#ffff00",
          "#ccff33",
          "#adff2f",
          "#5cd65c",
          "#33cc69",
          "#33cc9e",
        ];
      },
    },
    colorEnd: {
      type: String,
      default: "#5cd65c",
    },
    colorReversed: {
      type: Boolean,
      default: false,
    },
    colorStart: {
      type: String,
      default: "#fc0303",
    },
    dark: {
      type: Boolean,
      default: false,
    },
    darkColor: {
      type: String,
      default: "#18192C",
    },
    gradient: {
      type: Boolean,
      default: false,
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
    msBeforeMount: {
      type: Number,
      default: 400,
    },
    range: {
      type: Array,
      default() {
        // total must be 100
        return [10, 10, 10, 10, 10, 10, 10, 10, 10, 10];
      },
    },
    rounding: {
      type: Number,
      default: 1,
    },
    score: {
      type: Number,
      default: 0,
    },
    showPlusSign: {
      type: Boolean,
      default: false,
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
      chartParams: {
        x: 200,
        y: 200,
        radius: 133.33,
        lineWidth: 40,
        strokeStyle: "#fff",
      },
      isLoading: true,
      isTooltip: false,
      rotation: 1.5,
      up: this.min,
      speed: Number(this.animationSpeed),
    };
  },
  mounted() {
    this.isLoading = true;
    setTimeout(() => {
      this.drawGauge();
      this.isLoading = false;
    }, this.msBeforeMount);
  },
  computed: {
    colorSet() {
      if (this.gradient) {
        return this.gradientColors;
      }
      if (this.colorReversed) {
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        return this.colors.reverse();
      }
      return this.colors;
    },
    gradientColors() {
      return this.generateColorRange(
        this.colorStart,
        this.colorEnd,
        this.range.length
      );
    },
    canvas() {
      return this.$refs.customGaugeCanvas;
    },
    ctx() {
      return this.canvas.getContext("2d");
    },
    colorTheme() {
      if (this.dark) {
        return {
          recto: this.darkColor,
          verso: "white",
        };
      } else {
        return {
          recto: "white",
          verso: this.darkColor,
        };
      }
    },
    tickTypes() {
      const gap = this.max - this.min;
      return [
        {
          positions: this.createPositions(this.min, this.max, 1, 1),
          size: 153,
          lineWidth: 2,
        },
        {
          positions: this.createPositions(
            this.min + gap / gap / 2,
            this.max - gap / gap,
            1 + gap / gap,
            0.5
          ),
          size: 130,
          lineWidth: 1,
        },
        {
          positions: this.createPositions(
            this.min + gap / gap / 10,
            this.max - gap / gap,
            1 + gap / gap,
            0.1
          ),
          size: 120,
          lineWidth: 0.5,
        },
      ];
    },
    allTicks() {
      return this.tickTypes
        .map((tickType) =>
          tickType.positions.map((position) => Number(position.toFixed(1)))
        )
        .flat();
    },
  },

  updated() {
    this.drawGauge();
  },

  methods: {
    animate() {
      this.ctx.save();
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      const { x, y } = this.chartParams;
      let tempScore = this.up > this.score ? this.score : this.up;
      if (this.min > 0 && tempScore < this.min) {
        tempScore = this.min;
      }
      const pointerSize = 110;
      const initRotation = this.getGaugeRotation(0 + tempScore);
      const x2 =
        x + pointerSize * Math.sin(this.degreesToRadians(initRotation));
      this.ctx.clearRect(0, 0, 400, 400);
      const y2 =
        y + pointerSize * -1 * Math.cos(this.degreesToRadians(initRotation));

      this.drawRange();
      this.drawTicks();
      if (!this.hideMeasures) {
        this.drawMeasures();
      }

      this.drawHollow();
      this.drawScore(tempScore, 45, false);
      this.drawPointerCenter(20, this.getScoreColor(tempScore));
      this.drawPointer(
        x2,
        y2,
        pointerSize,
        this.getScoreColor(tempScore),
        tempScore
      );
      this.drawPointerDetails(1, x2, y2, "white");
      this.drawPointerCenter(10, this.getScoreColor(tempScore));
      if (this.up < this.score) {
        this.up += Number(this.acceleration) * this.speed;
        this.speed += Number(this.acceleration);
        requestAnimationFrame(this.animate);
      }
      this.ctx.restore();
    },
    drawGauge() {
      if (this.animated) {
        requestAnimationFrame(this.animate);
      } else {
        const { x, y } = this.chartParams;
        const pointerSize = 110;
        const initRotation = this.getGaugeRotation(this.score);
        const x2 =
          x + pointerSize * Math.sin(this.degreesToRadians(initRotation));
        this.ctx.clearRect(0, 0, 400, 400);
        const y2 =
          y + pointerSize * -1 * Math.cos(this.degreesToRadians(initRotation));
        this.drawRange();
        this.drawTicks();

        if (!this.hideMeasures) {
          this.drawMeasures();
        }
        this.drawHollow();
        this.drawScore(this.score, 45, false);
        this.drawPointerCenter(20, this.getScoreColor(this.score));
        this.drawPointer(
          x2,
          y2,
          pointerSize,
          this.getScoreColor(this.score),
          this.score
        );
        this.drawPointerDetails(1, x2, y2, "white");
        this.drawPointerCenter(10, this.getScoreColor(this.score));
      }
    },
    degreesToRadians(degrees) {
      return (degrees * Math.PI) / 180;
    },
    drawHollow() {
      const { x, y } = this.chartParams;
      this.ctx.beginPath();
      this.ctx.strokeStyle = this.colorTheme.recto;
      this.ctx.lineWidth = 1;
      this.ctx.arc(x, y, 112, 0, Math.PI * 2);
      this.ctx.fillStyle = this.colorTheme.recto;
      this.ctx.fill();
      this.ctx.stroke();
    },
    drawRange() {
      const { lineWidth, radius, strokeStyle, x, y } = this.chartParams;
      this.ctx.beginPath();
      this.ctx.lineWidth = lineWidth;
      this.ctx.strokeStyle = strokeStyle;
      this.ctx.arc(x, y, radius, 0, 0);
      let df = 2.36;
      for (let i = 0; i < this.range.length; i += 1) {
        this.ctx.beginPath();
        this.ctx.strokeStyle = this.colorSet[i];
        this.ctx.arc(
          x,
          y,
          radius,
          df,
          df + Math.PI * this.rotation * (this.range[i] / 100)
        );
        this.ctx.stroke();
        df += Math.PI * this.rotation * (this.range[i] / 100);
      }
    },
    drawTickType(tick, tickSize, lineWidth) {
      const { x, y } = this.chartParams;
      const rotation = this.getGaugeRotation(tick);
      const x2 = x + tickSize * Math.sin(this.degreesToRadians(rotation));
      const y2 = y + tickSize * -1 * Math.cos(this.degreesToRadians(rotation));
      this.ctx.lineWidth = lineWidth;
      this.ctx.strokeStyle = "rgb(60,60,60)";
      this.ctx.beginPath();
      this.ctx.moveTo(x, y);
      this.ctx.lineTo(x2, y2);
      this.ctx.stroke();
    },
    createPositions(min, max, incr, step) {
      const gap = max - min;
      const positions = [];
      for (let i = gap - 1 + incr; i >= 0; i -= step) {
        positions.push(i);
      }
      return positions;
    },
    drawTicks() {
      const type0 = this.tickTypes[0];
      const type1 = this.tickTypes[1];
      const draw = (index, position) => {
        this.drawTickType(
          position,
          this.tickTypes[index].size,
          this.tickTypes[index].lineWidth
        );
      };
      this.tickTypes[0].positions.forEach((position, i) => {
        if (type0.positions.length > 30) {
          if (i % 20 === 0) {
            draw(0, position);
          }
        } else {
          draw(0, position);
        }
      });

      this.tickTypes[1].positions.forEach((position, i) => {
        if (type1.positions.length > 300) {
          if (i % 10 === 0) {
            draw(1, position);
          }
        } else {
          draw(1, position);
        }
      });

      this.tickTypes[2].positions.forEach((position) => {
        if (type0.positions.length < 30) {
          draw(2, position);
        }
      });
    },
    drawPointer(x2, y2, size, _color, score) {
      const { x, y } = this.chartParams;
      const rotation = this.getGaugeRotation(score - this.min);

      x2 = x + size * Math.sin(this.degreesToRadians(rotation));
      y2 = y + size * -1 * Math.cos(this.degreesToRadians(rotation));

      const pointerWidth = 35;
      const gradient = this.ctx.createRadialGradient(x, y, 1, x2, y2, 75);
      gradient.addColorStop(0, "white");
      gradient.addColorStop(1, this.getScoreColor(score) || "grey");
      this.ctx.fillStyle = gradient;
      this.ctx.lineWidth = 1;
      this.ctx.strokeStyle = "grey";
      let angle = Math.atan2(y2 - y, x2 - x);
      this.ctx.save();
      this.ctx.beginPath();
      this.ctx.moveTo(x, y);
      this.ctx.lineTo(x2, y2);
      this.ctx.stroke();
      this.ctx.beginPath();
      this.ctx.moveTo(x2, y2);
      this.ctx.lineTo(
        x2 - size * Math.cos(angle - Math.PI / pointerWidth),
        y2 - size * Math.sin(angle - Math.PI / pointerWidth)
      );
      this.ctx.lineTo(
        x2 - size * Math.cos(angle + Math.PI / pointerWidth),
        y2 - size * Math.sin(angle + Math.PI / pointerWidth)
      );
      this.ctx.lineTo(x2, y2);
      this.ctx.lineTo(
        x2 - size * Math.cos(angle - Math.PI / pointerWidth),
        y2 - size * Math.sin(angle - Math.PI / pointerWidth)
      );
      this.ctx.closePath();
      this.ctx.fill();
      this.ctx.stroke();
      this.ctx.restore();
    },
    drawPointerDetails(_thickness, x2, y2) {
      const { x, y } = this.chartParams;
      this.ctx.lineWidth = 1;
      this.ctx.strokeStyle = "transparent";
      this.ctx.beginPath();
      this.ctx.moveTo(x, y);
      this.ctx.lineTo(x2, y2);
      this.ctx.stroke();
    },
    drawPointerCenter(radius) {
      const { x, y } = this.chartParams;
      this.ctx.beginPath();
      this.ctx.strokeStyle = "white";
      this.ctx.lineWidth = 1;
      const gradient = this.ctx.createRadialGradient(
        x,
        y,
        1,
        x + 2,
        y - radius / 2,
        radius
      );
      gradient.addColorStop(0, "white");
      gradient.addColorStop(1, "grey");
      this.ctx.arc(x, y, radius, 0, Math.PI * 2);
      this.ctx.fillStyle = gradient;
      this.ctx.fill();
      this.ctx.stroke();
    },
    drawIndividualMeasure(x, y, position) {
      const rotation = this.getGaugeRotation(position);
      const x2 = x + 170 * Math.sin(this.degreesToRadians(rotation));
      const y2 = y + 170 * -1 * Math.cos(this.degreesToRadians(rotation));
      this.ctx.strokeStyle = this.colorTheme.verso;
      this.ctx.font = "20px Arial";
      this.ctx.fillStyle = this.colorTheme.verso;
      this.ctx.textAlign = "center";
      let text = position;
      text = position + this.min;
      this.ctx.fillText(text, x2, y2 + 8);
    },
    drawMeasures() {
      const { x, y } = this.chartParams;
      return this.tickTypes[0].positions.forEach((position, i) => {
        if (this.tickTypes[0].positions.length > 20) {
          if (i % 20 === 0) {
            this.drawIndividualMeasure(x, y, position);
          }
        } else {
          this.drawIndividualMeasure(x, y, position);
        }
      });
    },
    drawScore(score) {
      const { x, y } = this.chartParams;
      this.ctx.strokeStyle = this.getScoreColor(score) || "grey";
      this.ctx.font = `700 45px Arial`;
      this.ctx.fillStyle = this.getScoreColor(score) || "grey";
      this.ctx.textAlign = "center";
      let sign;
      if (this.showPlusSign) {
        if (score > 0) {
          sign = "+";
        } else if (score < 0) {
          sign = " ";
        } else {
          sign = null;
        }
      }

      this.ctx.fillText(
        `${sign ? sign : ""}${score.toFixed(this.rounding)}`,
        sign ? x - 10 : x,
        y + 120
      );
    },
    getScoreColor(score) {
      const range = this.range;
      let colorSteps = [];
      let universalScale = [];
      let accumulator = 0;
      for (let i = 0; i < range.length; i += 1) {
        colorSteps.push(range[i] + accumulator);
        universalScale.push({
          color: this.colorSet[i],
          step: range[i] + accumulator,
        });
        accumulator += range[i];
      }

      const color = {};

      universalScale.forEach((scale, i) => {
        color[scale.step] = this.colorSet[i];
      });

      let gap;
      if (this.min > 0) {
        gap = this.max - this.min;
      } else if (this.min < 0) {
        gap = this.max + Math.abs(this.min);
      } else {
        gap = this.max;
      }

      const closest =
        universalScale.find((el) => {
          return el.step > ((score - this.min) / gap) * 100;
        }) || universalScale[universalScale.length - 1];

      return closest.color || this.colorSet[0];
    },
    getGaugeRotation(value = 0) {
      return -135 + value * (270 / (this.max - this.min));
    },
    reinit() {
      this.up = this.min;
      this.speed = Number(this.animationSpeed);
    },
    generateColorRange(color1, color2, steps) {
      function rgbToHex(r, g, b) {
        return (
          "#" + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1)
        );
      }

      function hexToRgb(hex) {
        let result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
        return result
          ? {
              r: parseInt(result[1], 16),
              g: parseInt(result[2], 16),
              b: parseInt(result[3], 16),
            }
          : null;
      }

      const colorSet = [];

      colorSet.push(color1);

      let color1Rgb = hexToRgb(color1);
      let color2Rgb = hexToRgb(color2);

      let rInc = Math.round((color2Rgb.r - color1Rgb.r) / (steps + 1));
      let gInc = Math.round((color2Rgb.g - color1Rgb.g) / (steps + 1));
      let bInc = Math.round((color2Rgb.b - color1Rgb.b) / (steps + 1));

      for (let i = 0; i < steps; i++) {
        color1Rgb.r += rInc;
        color1Rgb.g += gInc;
        color1Rgb.b += bInc;

        colorSet.push(rgbToHex(color1Rgb.r, color1Rgb.g, color1Rgb.b));
      }
      colorSet.push(color2);
      return colorSet;
    },
  },
});
</script>

<style lang="scss" scoped>
.gauge {
  &__canvas {
    height: 300px;
    border-radius: 50%;
  }
  &__container {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    width: fit-content;
    position: relative;
  }
  &__tooltip-trap {
    height: 170px;
    left: 50%;
    position: absolute;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 170px;
    border-radius: 50%;
  }
  &__tooltip {
    background: white;
    border-radius: 6px;
    bottom: -20px;
    box-shadow: 0px 10px 20px -5px grey;
    height: fit-content;
    left: 50%;
    max-width: 250px;
    position: absolute;
    transform: translateX(-50%);
    width: fit-content;
  }
}
.test {
  display: block;
  height: 100px;
  width: 100px;
  border-radius: 6px;
}
</style>
