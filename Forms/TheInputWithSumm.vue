<template>
  <div class="input-wrapp text-field text-field_floating">
    <input
      class="text-field__input"
      :name="name"
      type="text"
      :id="id"
      placeholder=""
     v-model="inputValue"
      @input="onInput($event)"
      />
    <span class="suffix">{{ suffix }}</span>
    <label class="text-field__label" :for="id">{{ label }}</label>
    <span v-if="error" class="error-text">{{ error }}</span>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const model = defineModel();
const props = defineProps({
  id: String,
  name: String,
  label: String,
  suffix: {
    type: String,
    default: "₽",
  },
  error: String,
});

const inputValue = ref("");

watch(
  () => model.value,
  (val) => {
    if (val !== inputValue.value.replace(/\s/g, "")) {
      inputValue.value = formatCurrency(val);
    }
  },
  { immediate: true }
);

function formatCurrency(value) {
  if (!value) return "";
  const digits = value.toString().replace(/\D/g, "");
  if (!digits) return "";
  return digits.replace(/\B(?=(\d{3})+(?!\d))/g, " ");
}

function onInput(e) {
  const raw = e.target.value.replace(/\D/g, "");
  inputValue.value = formatCurrency(raw);
  model.value = raw;
}
</script>

<style scoped>
.input-wrapp{
  flex: 1;
}
.text-field__input {
  width: 100%;
  height: calc(3.5rem + 2px);
  line-height: 1.25;
  padding: 1rem 2.5rem 1rem 0.75rem;
  background: #f3f4f7;
  border: 1px solid #f3f4f7;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.2s;
  position: relative;
}

.text-field__input:hover {
  border-color: #f90042;
}

.suffix {
  position: absolute;
  right: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  color: #888;
  pointer-events: none;
}

.text-field_floating {
  position: relative;
}
.text-field__label {
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
.text-field__input:focus,
.text-field__input:not(:placeholder-shown) {
  padding-top: 1.625rem;
  padding-bottom: 0.625rem;
}

.text-field__input:focus ~ .text-field__label,
.text-field__input:not(:placeholder-shown) ~ .text-field__label {
  transform: translateY(-1.5rem) scale(0.85);
  font-size: 0.875rem;
  opacity: 0.65;
}
</style>
