<script setup>
import { ref, watch, computed } from 'vue'

const props = defineProps({
  array: {
    type: Array,
    default: () => [],
  },
  label: String,
  id: String,
  name: {
    type: String,
    required: true,
  },
  placeholder: String,
  current: {
    type: String,
    default: '',
  },
  error: String,
})

const emit = defineEmits(['select', 'citynames'])

const value = ref('')
const hints = ref([])
const showResult = ref(false)

function filterArray() {
  hints.value = props.array.filter((item) =>
    item.name.toLowerCase().includes(value.value.toLowerCase())
  )
}

function onInput() {
  emit('citynames', value.value)

  if (!value.value) {
    showResult.value = false
    return
  }

  filterArray()
  showResult.value = true
}

function selectHint(item) {
  if (!item) return

  value.value = item.name
  emit('select', item)
  hints.value = []
  showResult.value = false
}

// следим за обновлением текущего значения из props
watch(
  () => props.current,
  (val) => {
    if (val) {
      value.value = val
    }
  },
  { immediate: true }
)

// если массив пришел заново — фильтруем снова
watch(
  () => props.array,
  () => {
    filterArray()
  },
  { deep: true }
)
</script>

<template>
  <div class="input-wrapp text-field text-field_floating">
    <input
      class="text-field__input"
      @input="onInput"
      type="text"
      :name="name"
      :id="id"
      v-model="value"
      placeholder=" " />
    <label class="text-field__label" :for="id">{{ label }}</label>
    <span class="arrow"><i class="bi bi-chevron-down"></i></span>

    <div class="options" v-if="showResult">
      <li
        v-for="(item, index) in hints"
        :key="index"
        @click="selectHint(item)">
        {{ item.name }}
      </li>
    </div>

    <span v-if="error" class="error-text">{{ error }}</span>
  </div>
</template>




<style scoped>
.text-field_floating {
  position: relative;
}
.selected {
  position: relative;
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.options {
  position: absolute;
  top: calc(100% + 6px);
  left: 0;
  right: 0;
  max-height: 400px;
  overflow-y: auto;
  background: white;
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.07);
  z-index: 10000;

  list-style: none;
  margin: 0;
  padding: 5px 0;
}
.options li {
  padding: 10px 15px;
  transition: background 0.2s;
}
.options li:hover {
  background: #f5f5f5;
}
.arrow {
  position: absolute;
  right: 12px;
  top: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  pointer-events: none;
  transition: transform 0.3s ease;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.text-field_floating .text-field__input {
  width: 100%;
  /* max-width: 200px; */
  height: calc(3.5rem + 2px);
  line-height: 1.25;
  padding: 1rem 0.75rem;
  background: #f3f4f7;
  border: 1px solid #f3f4f7;
  border-radius: 8px;
  transition: border-color 0.2s;
  font-size: 16px;
  cursor: pointer;
}
.text-field_floating .text-field__input:hover {
  border-color: #f90042;
}

.text-field_floating .text-field__label {
  position: absolute;
  top: 50%;
  left: 0.75rem;
  transform: translateY(-50%);
  background: #f3f4f7;
  padding: 0 0.25rem;
  font-size: 1rem;
  pointer-events: none;
  transition: transform 0.15s ease-in-out, font-size 0.15s ease-in-out,
    opacity 0.15s ease-in-out;
}

.text-field_floating .text-field__input::placeholder {
  color: transparent;
  transition: color 0.15s ease-in-out;
}

.text-field_floating .text-field__input:focus::placeholder {
  color: #bbb;
}

.text-field_floating .text-field__input:focus,
.text-field_floating .text-field__input:not(:placeholder-shown) {
  padding-top: 1.625rem;
  padding-bottom: 0.625rem;
}

.text-field_floating .text-field__input:focus ~ .text-field__label,
.text-field_floating
  .text-field__input:not(:placeholder-shown)
  ~ .text-field__label {
  transform: translateY(-1.5rem) scale(0.85);
  font-size: 0.875rem;
  opacity: 0.65;
  background: transparent;
  padding: 0 0.25rem;
}
</style>
