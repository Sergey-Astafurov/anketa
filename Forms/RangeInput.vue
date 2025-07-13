<script setup>
import { computed } from "vue";
import VueSlider from "vue-slider-component";
import "vue-slider-component/theme/default.css";

const props = defineProps({
  ageRange: {
    type: Array,
  }
});

const emit = defineEmits(["update:ageRange"]);

const localAgeRange = computed({
  get() {
    return props.ageRange
  },
  set(val) {
    emit("update:ageRange", val);
  }
});
</script>

<template>
  <div>
    <div class="h5 mb-2">Предпочтительный возраст клиента</div>
    <label class="mb-2">Возраст: {{ localAgeRange[0] }} – {{ localAgeRange[1] }}</label>
    <vue-slider
      v-model="localAgeRange"
      :min="18"
      :max="66"
      :dot-size="10"
      :tooltip="'always'"
      :marks="[18, 25, 35, 45, 55, 66]"
      :lazy="true"
      :enable-cross="false"
    />
  </div>
</template>

<style>
.vue-slider {
  height: 30px;
}
.vue-slider-rail {
  height: 10px;
}
.vue-slider-process {
  height: 6px;
  background: #f90042;
}
.vue-slider-dot-tooltip-inner-top::after {
  border-top-color: #f90042;
}
.vue-slider-dot-tooltip-inner {
  background: #f90042;
}
</style>
