<template>
  <div class="color-options d-flex flex-column">
    <h5 class="h5 mb-1">Цвет волос</h5>
    <div class=" options">
      <div
        v-for="color in hairColors"
        :key="color.value"
        class="color-circle"

        :class="{ selected: selectedColor === color.value }"
        :style="{ backgroundColor: color.hex }"
       @click="selectedColor = color.value"
        :title="color.label"></div>
      </div>
      <span class="select__color">{{ selectedLabel }}</span>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const props = defineProps({
  hairColors: Array,
})
const selected = defineModel('selected')
const selectedColor = computed({
  get: () => selected.value,
  set: (val) => {
    selected.value = val
  }
})

const selectedLabel = computed(() => {
  const match = props.hairColors.find(c => c.value === selectedColor.value);
  return match ? match.label : '';
});
</script>

<style scoped>
.color-options {
  display: flex;
  gap: 10px;
}
.color-circle {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  cursor: pointer;
  border: 2px solid transparent;
  transition: 0.2s;
}
.color-circle.selected {
  border-color: #f90042;
}
.options{
  display: flex;
  gap: 15px;

}
.select__color{
  text-align: center;
  font-size: 20px;
  transition: all .3s ease;
}
</style>
