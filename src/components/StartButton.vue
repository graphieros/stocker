<template>
  <Clicker
    @click="$emit('togglePlayState')"
    :background="playState === 1 ? '#5C9A54' : '#ff9933'"
    textColor="white"
    :borderRadius="8"
    bold
    reflection
    :reflectionIntensity="0.5"
    fab
    :tooltipHtml="playState === 1 ? 'Play' : 'Pause'"
  >
    <svg
      v-if="playState === 1"
      style="width: 24px; height: 24px"
      viewBox="0 0 24 24"
    >
      <path fill="currentColor" d="M8,5.14V19.14L19,12.14L8,5.14Z" />
    </svg>
    <svg v-else style="width: 24px; height: 24px" viewBox="0 0 24 24">
      <path fill="currentColor" d="M14,19H18V5H14M6,19H10V5H6V19Z" />
    </svg>
  </Clicker>
</template>

<script>
import Vue from "vue";
import Clicker from "../components/atoms/Clicker.vue";
export default Vue.extend({
  name: "StartButton",
  props: {
    playState: {
      type: Number,
    },
  },
  components: { Clicker },
  computed: {},
  data() {
    return {};
  },
  methods: {},
});
</script>
