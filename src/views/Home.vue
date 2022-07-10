<template>
  <div :class="{ stocker: true, 'stocker--pause': playState === 1 }">
    <div class="stocker__actions__start">
      <label for="range" style="color: white"
        >Speed: {{ settings.speed }}</label
      >
      <input
        id="range"
        type="range"
        min="10"
        max="1000"
        v-model="settings.speed"
      />
      <Clicker
        :disabled="playState === 1"
        @click="buy(settings.tradeAmount)"
        textColor="white"
        fab
        reflection
        :reflectionIntensity="0.5"
      >
        <svg style="width: 24px; height: 24px" viewBox="0 0 24 24">
          <path
            fill="currentColor"
            d="M20 14H14V20H10V14H4V10H10V4H14V10H20V14Z"
          />
        </svg>
      </Clicker>
      <StartButton @togglePlayState="togglePlayState" :playState="playState" />
      <Clicker
        :disabled="playState === 1"
        @click="sell(settings.tradeAmount)"
        textColor="white"
        fab
        reflection
        :reflectionIntensity="0.5"
      >
        <svg style="width: 24px; height: 24px" viewBox="0 0 24 24">
          <path fill="currentColor" d="M20 14H4V10H20" />
        </svg>
      </Clicker>
    </div>
    <div class="stocker__control__main">
      <div class="stocker__control__gauge">
        <div class="stocker__control__gauge-wrapper">
          <Gauge
            animated
            :min="0"
            :max="100"
            :score="settings.krachRisk"
            dark
            :size="200"
            colorReversed
          />
          <div class="stocker__control__gauge__title">Krach risk</div>
        </div>
        <div class="stocker__control__gauge-wrapper">
          <Gauge
            animated
            :min="0"
            :max="20"
            :score="settings.marketTrend"
            dark
            :size="200"
          />
          <div class="stocker__control__gauge__title">Market trend</div>
        </div>
      </div>
      <LineChart
        :dataset="settings.stockHistory"
        :average="settings.stockAverageHistory"
        :wallet="wallet"
        :currentStockPrice="settings.stockValue"
      />
    </div>
  </div>
</template>

<script>
import Clicker from "../components/atoms/Clicker.vue";
import Gauge from "../components/charts/Gauge.vue";
import LineChart from "../components/charts/LineChart.vue";
import StartButton from "../components/StartButton.vue";
export default {
  name: "Home",
  components: { Clicker, Gauge, LineChart, StartButton },
  data() {
    return {
      gameTimeout: undefined,
      hide: true,
      playState: 1,
      currentTime: 0,
      settings: {
        growth: 1,
        krachRisk: 0,
        marketTrend: 1,
        speed: 100,
        stockAverageHistory: [],
        stockAverageHistory2: [],
        stockHistory: [],
        stockIncr: 0,
        stockValue: 0,
        tension: 0,
        tradeAmount: 1,
        trend: 1,
      },
      wallet: {
        cash: 10,
        stocks: 0,
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
    buy(stocks) {
      if (this.wallet.cash >= stocks * this.settings.stockValue) {
        this.wallet.stocks += stocks;
        this.wallet.cash -= stocks * this.settings.stockValue;
        this.settings.krachRisk += Math.random() * 10;
        this.settings.marketTrend += Math.random();
      }
    },
    sell(stocks) {
      if (this.wallet.stocks >= stocks) {
        this.wallet.cash += stocks * this.settings.stockValue;
        this.wallet.stocks -= stocks;
        this.settings.krachRisk -= Math.random();
        this.settings.marketTrend -= Math.random() * 1.2;
      }
    },
    floorStock() {
      if (this.settings.stockValue < 0) {
        this.settings.stockValue = 0;
      }
    },
    gameLoop() {
      if (this.playState === 0) {
        this.currentTime += 1;
        if (this.currentTime === 30) {
          this.currentTime = 0;
          this.krachRisk += Math.random() * 2;
        }
        const die = Math.random();
        if (die > 0.75) {
          this.settings.tension += die;
        } else {
          this.settings.tension -= die;
          if (this.settings.tension <= 0) {
            this.settings.tension = 0;
          }
        }
        if (this.settings.tension > 10) {
          this.settings.krachRisk += 1;
        }

        if (this.isTrendPositive(this.settings.tension)) {
          this.settings.stockIncr = this.settings.tension;
          this.updateMarketTrend();
        } else {
          this.settings.stockIncr = -this.settings.tension;
        }

        this.settings.krachRisk -= Math.random() * 0.01;
        if (this.settings.krachRisk <= 0) {
          this.settings.krachRisk = 0;
        }
        const krachDie = Math.random();
        if (this.settings.krachRisk > 50 && this.currentTime === 30) {
          if (krachDie > 0.2) {
            this.settings.stockValue /= 1 + Math.random() * 10;
          }
        } else {
          this.updateStockValue(this.settings.stockIncr);
          this.krachRisk -= Math.random() * 5;
        }
        if (this.settings.krachRisk >= 100) {
          this.settings.krachRisk = 0;
          this.settings.stockValue /= 1 + Math.random() * 10;
        }
        clearTimeout(this.gameTimeout);
        this.gameTimeout = setTimeout(this.gameLoop, this.settings.speed);
      }
    },
    isTrendPositive(tension) {
      const trend = Math.random();
      return tension > trend;
    },
    togglePlayState() {
      this.playState === 1 ? (this.playState = 0) : (this.playState = 1);
    },
    updateGrowth(rand) {
      this.settings.growth += rand;
    },
    updateMarketTrend() {
      const die = Math.random();

      if (die > 0.95) {
        this.settings.marketTrend += 1;
        this.settings.krachRisk += 0.01;
      } else {
        if (Math.random() > 0.6) {
          this.settings.marketTrend += die;
          this.settings.krachRisk += 0.01;
          if (this.settings.marketTrend >= 20) {
            this.settings.marketTrend /= Math.random() * 10;
          }
        } else {
          this.settings.marketTrend -= die;
        }
      }
      if (this.settings.marketTrend >= 20) {
        this.settings.marketTrend /= Math.random() * 10;
        this.settings.stockValue *= 1.01;
      }
      if (this.settings.marketTrend <= 0) {
        this.settings.marketTrend = 0;
        this.settings.stockValue *= 0.99;
      }
    },
    updateStockHistory() {
      this.settings.stockHistory.push(this.settings.stockValue);
      this.settings.stockHistory = this.settings.stockHistory.slice(-365);
      const average =
        [...this.settings.stockHistory].slice(-90).reduce((a, b) => a + b, 0) /
        90;
      this.settings.stockAverageHistory.push(average);
      this.settings.stockAverageHistory =
        this.settings.stockAverageHistory.slice(-365);
    },
    updateStockValue(incr) {
      this.settings.stockValue +=
        Math.random() * (this.settings.marketTrend / 2) * incr -
        Math.random() * incr;
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
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: center;
      gap: 12px;
      bottom: 36px;
      left: 50%;
      position: fixed;
      transform: translateX(-50%);
    }
  }
  &__control {
    &__gauge {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
    &__gauge-wrapper {
      position: relative;
    }
    &__gauge__title {
      position: absolute;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
    }
    &__main {
      display: flex;
      flex-direction: row;
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
