<template>
  <div class="field">
    <h5 class="h5 mb-1">Фигура</h5>
    <div class="figure-options">
      <label
        v-for="item in figures"
        :key="item.value"
        class="figure-option"
        :class="{ selected: model === item.value }"
      >
        <input
          type="radio"
          v-model="model"
          :value="item.value"
          class="hidden"
        />
        {{ item.label }}
      </label>
    </div>
  </div>
</template>



<script setup>
import { ref } from 'vue'
const model = defineModel()
const props = defineProps({
  figures:{
    type: Array,
  }

})
const figure = ref(null)

</script>

<style scoped>
.hidden{
  display: none;
}
.figure-options {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 8px;
}

.figure-option {
  padding: 10px 14px;
  width: 120px;
  text-align: center;
  border: 1px solid #f3f4f7;
  border-radius: 8px;
  cursor: pointer;
  user-select: none;
  font-size: 16px;
  transition: 0.2s;
  background-color: #f3f4f7;
}

.figure-option:hover {
  border-color: #f90042;
}

.figure-option.selected {
  border-color: #f90042;
  background-color: #f90042;
  color: #fff;
}
</style>
