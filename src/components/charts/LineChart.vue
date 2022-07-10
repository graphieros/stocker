<template>
  <svg class="chart" viewBox="0 0 370 350">
    <g v-for="(line, i) in lines" :key="`line_${i}`">
      <g>
        <line :x1="line.x1" :y1="line.y1" :x2="line.x2" :y2="line.y2" />
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
        r="2"
        :cx="lines[lines.length - 1]?.x2 || 0"
        :cy="lines[lines.length - 1]?.y2 || 0"
        fill="rgb(130,130,130)"
      />
    </g>
  </svg>
</template>

<script>
import Vue from "vue";
export default Vue.extend({
  name: "LineChart",
  components: {},
  props: {
    dataset: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  data() {
    return {
      svgHeight: 420,
    };
  },
  computed: {
    height() {
      if (this.max < 0) {
        return 400;
      } else if (this.max > 400) {
        return 400;
      }
      return this.max;
    },
    min() {
      return Math.min(...this.dataset);
    },
    max() {
      return Math.max(...this.dataset);
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
.chart-foster-parent {
  display: flex;
  padding-bottom: 10px;
  position: relative;
  width: 100%;
}

line {
  cursor: crosshair;
  stroke-width: 1;
  stroke: rgb(87, 87, 87);
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
  background: radial-gradient(rgba(0, 0, 0, 0.226), rgba(0, 0, 0, 0.705));
  border-radius: 3px;
  // border: 1px solid rgb(46, 46, 46);
  // border: 1px solid rgba(33, 149, 243, 0.336);
  box-shadow: 0 20px 40px -20px black;
  margin: 10px;
  padding: 20px;
  width: 400px;
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
