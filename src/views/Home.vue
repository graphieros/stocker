<template>
  <div :class="{ stocker: true, 'stocker--pause': playState === 1 }">
    <Gauge v-if="!hide" animated :min="0" :max="10" :score="5" dark />

    <div class="stocker__actions__start">
      <StartButton @togglePlayState="togglePlayState" :playState="playState" />
    </div>
    <div class="stocker__control__main">
      {{ settings.stockValue }}
      <LineChart :dataset="settings.stockHistory" />
    </div>
  </div>
</template>

<script>
import Gauge from "../components/charts/Gauge.vue";
import LineChart from "../components/charts/LineChart.vue";
import StartButton from "../components/StartButton.vue";
export default {
  name: "Home",
  components: { Gauge, LineChart, StartButton },
  data() {
    return {
      gameTimeout: undefined,
      hide: true,
      playState: 1,
      settings: {
        stockHistory: [],
        stockIncr: 0,
        stockValue: 0,
        tension: 0,
        trend: 1,
      },
    };
  },
  watch: {
    playState: {
      handler(val) {
        if (val === 0) {
          this.gameLoop();
        }
      },
    },
  },
  methods: {
    floorStock() {
      if (this.settings.stockValue < 0) {
        this.settings.stockValue = 0;
      }
    },
    gameLoop() {
      if (this.playState === 0) {
        this.settings.tension = Math.random();
        if (this.isTrendPositive(this.settings.tension)) {
          this.settings.stockIncr = this.settings.tension;
        } else {
          this.settings.stockIncr = -this.settings.tension;
        }
        this.updateStockValue(this.settings.stockIncr);
        clearTimeout(this.gameTimeout);
        this.gameTimeout = setTimeout(this.gameLoop, 100);
      }
    },
    isTrendPositive(tension) {
      const trend = Math.random();
      return tension > trend;
    },
    togglePlayState() {
      this.playState === 1 ? (this.playState = 0) : (this.playState = 1);
    },
    updateStockHistory() {
      this.settings.stockHistory.push(this.settings.stockValue);
      this.settings.stockHistory = this.settings.stockHistory.slice(-365);
    },
    updateStockValue(incr) {
      this.settings.stockValue += Math.random() * incr;
      this.floorStock();
      this.updateStockHistory();
    },
  },
};
</script>

<style lang="scss">
* {
  font-family: "Roboto", sans-serif;
  font-family: "Ubuntu Mono", monospace;
}
body {
  background: black;
}
.stocker {
  height: calc(100vh - 16px);
  border-radius: 36px;
  width: 100%;
  display: block;
  position: relative;
  background: #18192c;
  &__actions {
    &__start {
      bottom: 36px;
      left: 50%;
      position: fixed;
      transform: translateX(-50%);
    }
  }
  &__control {
    &__main {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      width: calc(100% - 72px);
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: white;
    }
  }
  &--pause {
    background: radial-gradient(at bottom, transparent, #18192c);
  }
}
</style>
