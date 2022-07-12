<template>
  <div class="chart__wrapper">
    <div v-if="showLegend" class="chart__info__cash">
      Cash: {{ Math.round(wallet.cash).toLocaleString() }}
    </div>
    <div v-if="showLegend" class="chart__info__stocks">
      Stocks: {{ wallet.stocks }}
    </div>
    <div v-if="showLegend" class="chart__info__price">
      Stock price: {{ currentStockPrice.toFixed(1).toLocaleString() }}
    </div>
    <svg
      class="chart"
      viewBox="0 0 370 350"
      :style="`height:${height}px; width:${width}px`"
    >
      <g v-for="(line, i) in lines" :key="`line_${i}`">
        <g v-if="!hideDataset">
          <line
            class="direct"
            :x1="line.x1"
            :y1="line.y1 && line.y1 !== Infinity ? line.y1 : 0"
            :x2="line.x2"
            :y2="line.y2 && line.y2 !== Infinity ? line.y2 : 0"
            :style="colorDataset ? `stroke:${colorDataset}` : ''"
          />
        </g>
        <line
          class="line-segment"
          v-if="i % timeSegment === 1"
          :x1="line.x1"
          :y1="20"
          :x2="line.x1"
          :y2="350"
        />
        <circle
          v-if="!hideDataset"
          r="2"
          :cx="lines[lines.length - 1].x2 ? lines[lines.length - 1].x2 : 0"
          :cy="lines[lines.length - 1].y2 ? lines[lines.length - 1].y2 : 0"
          :fill="colorDataset ? colorDataset : 'rgb(130, 130, 130)'"
        />
      </g>
      <line class="line-segment" :x1="0" :y1="350" :x2="360" :y2="350"></line>
      <line class="line-segment" :x1="0" :y1="20" :x2="360" :y2="20"></line>
      <g v-for="(line, i) in averageLines" :key="`average_line_${i}`">
        <g v-if="!hideAverage">
          <line
            class="chart__average-line"
            :x1="line.x1"
            :y1="line.y1 && line.y1 !== Infinity ? line.y1 : 0"
            :x2="line.x2"
            :y2="line.y2 && line.y2 !== Infinity ? line.y2 : 0"
          />
        </g>
      </g>
    </svg>
  </div>
</template>

<script>
import Vue from "vue";
export default Vue.extend({
  name: "LineChart",
  components: {},
  props: {
    average: {
      type: Array,
      default() {
        return [];
      },
    },
    currentStockPrice: {
      type: Number,
      default: 0,
    },
    colorDataset: {
      type: String,
      default: undefined,
    },
    dataset: {
      type: Array,
      default() {
        return [];
      },
    },
    height: {
      type: Number,
      default: null,
    },
    hideDataset: {
      type: Boolean,
      default: false,
    },
    hideAverage: {
      type: Boolean,
      default: false,
    },
    showLegend: {
      type: Boolean,
      default: true,
    },
    wallet: {
      type: Object,
      default() {
        return {
          cash: 0,
          stocks: 0,
        };
      },
    },
    width: {
      type: Number,
      default: null,
    },
  },
  data() {
    return {
      svgHeight: 420,
    };
  },
  computed: {
    min() {
      return Math.min(...this.dataset);
    },
    max() {
      return Math.max(...this.dataset);
    },
    averageLines() {
      return this.averagePlots.map((plot, i) => {
        return {
          x1: plot.x || 0,
          y1: plot.y || 0,
          x2: this.averagePlots[i + 1]
            ? this.averagePlots[i + 1].x
            : plot.x || 0,
          y2: this.averagePlots[i + 1]
            ? this.averagePlots[i + 1].y
            : plot.y || 0,
        };
      });
    },
    lines() {
      return this.plots.map((plot, i) => {
        return {
          x1: plot.x || 0,
          y1: plot.y || 0,
          x2: this.plots[i + 1] ? this.plots[i + 1].x : plot.x || 0,
          y2: this.plots[i + 1] ? this.plots[i + 1].y : plot.y || 0,
        };
      });
    },
    averagePlots() {
      return this.average.map((val, i) => {
        return {
          x: i,
          y: this.normalize(val),
        };
      });
    },
    plots() {
      return this.dataset.map((val, i) => {
        return {
          x: i,
          y: this.normalize(val),
        };
      });
    },
    timeSegment() {
      return Math.floor(365 / 12);
    },
  },
  methods: {
    normalize(val) {
      const norm = (val - this.min) / (this.max - this.min);
      const calculated = Math.abs(1 - norm) * (this.svgHeight / 1.6);
      return calculated + 80;
    },
  },
});
</script>

<style lang="scss" scoped>
line {
  cursor: crosshair;
  stroke: rgb(87, 87, 87);
}

.chart {
  &__average-line {
    stroke: #ff9933;
    stroke-width: 1;
  }
  &__info {
    &__cash,
    &__stocks {
      font-size: 0.8rem;
    }
  }
  &__info__cash {
    position: absolute;
    top: 18px;
    left: 30px;
    z-index: 1;
  }
  &__info__stocks {
    position: absolute;
    top: 32px;
    left: 30px;
    z-index: 1;
  }
  &__info__price {
    position: absolute;
    top: 18px;
    right: 30px;
    z-index: 1;
    text-align: right;
    font-size: 1.2rem;
  }
  &__wrapper {
    position: relative;
  }
}

.direct {
  stroke: rgba(255, 255, 255, 0.445);
}

.line-up,
.line-down,
.line-neutral {
  stroke-width: 3px;
  stroke-linecap: round;
  stroke-linejoin: round;
}

.line-up {
  stroke: #4caf50;
}

.line-down {
  stroke: #f65555;
}

.line-neutral-up,
.line-neutral-down {
  stroke-width: 2px;
}

.line-neutral-up {
  stroke: #2d772f;
}

.line-neutral-down {
  stroke: #a33b3b;
}

.line-trend {
  cursor: initial;
  stroke-dasharray: 2, 2;
  stroke-width: 3;
  transition: all 0.9s ease-in-out;
}

.up {
  stroke: #4caf50;
}

.down {
  stroke: #f65555;
}
.line-segment {
  stroke-width: 0.5;
  stroke: rgb(54, 54, 54);
}
.line-average {
  stroke-width: 2;
  stroke: rgb(152, 209, 255);
  // stroke-dasharray: 2, 2;
}
svg.chart {
  // background: linear-gradient(to top, rgba(255, 255, 255, 0.2), transparent);
  border-radius: 3px;
  // box-shadow: 0 20px 40px -20px black;
  margin: 10px;
  padding: 20px 20px 20px 20px;
  width: 400px;
  position: relative;
}
circle {
  transition: r 0.1s ease-in-out;
  cursor: crosshair;
}
circle:hover,
.circle-selected {
  fill: transparent;
  r: 5; /** that's not a css problem guys */
  stroke-width: 1;
  stroke: white;
}

.time-series {
  fill: grey;
  font-size: 0.6em;
  transform: rotate(-0deg);
}
</style>
