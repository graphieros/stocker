<template>
  <div class="simple-thermometer">
    <label
      for="base"
      :style="labelStyle"
      style="margin-top: -30px; font-size: 0.6rem"
      >{{ label }}</label
    >
    <input v-if="label" id="base" style="visibility: hidden" />
    <div class="simple-thermometer__base"></div>
    <div class="simple-thermometer__done" :style="doneStyle"></div>
  </div>
</template>

<script>
import Vue from "vue";
export default Vue.extend({
  name: "SimpleThermometer",
  props: {
    color: {
      type: String,
      default: "#ccc",
    },
    dark: {
      type: Boolean,
      default: false,
    },
    dataset: {
      type: Object,
      default() {
        return {
          done: 0,
          max: 1,
          ratio: 0,
        };
      },
    },
    gradient: {
      type: Boolean,
      default: true,
    },
    label: {
      type: String,
      default: "",
    },
    rounding: {
      type: Number,
      default: 1,
    },
  },
  data() {
    return {};
  },
  computed: {
    doneStyle() {
      const background = this.gradient
        ? `radial-gradient(white, ${this.color})`
        : this.color;
      return `
            background: ${background};
            width: ${this.dataset.ratio}%;
        `;
    },
    labelStyle() {
      return `
            margin-top: -30px;
            font-size: 0.6rem;
            color:${this.dark ? "white" : "black"}
        `;
    },
  },
  methods: {},
});
</script>

<style lang="scss" scoped>
.simple-thermometer {
  align-items: center;
  display: flex;
  height: fit-content;
  position: relative;
  width: 100%;
  margin: 0 auto;
  &__base,
  &__done {
    height: 10px;
    border-radius: 10px;
    position: absolute;
  }
  &__base {
    width: 100%;
    background: #ccc;
  }
  &__done {
    box-shadow: 3px 0px 6px -2px rgba(0, 0, 0, 0.3);
  }
}
label {
  color: white !important;
}
</style>
