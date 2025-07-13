<template>
  <div class="input-wrapp text-field text-field_floating">
    <textarea
      class="text-field__input"
      rows="10"
      cols="10"
      :name="name"
      type="text"
      :id="id"
      v-model="model"
      placeholder="" />
    <label class="text-field__label" :for="id">{{label}}</label>
  </div>
</template>

<script setup>
const model = defineModel()
const props = defineProps({
  id: String,
  name: String,
  label: String,
})
</script>

<style scoped>
.text-field_floating {
  position: relative;
}

.text-field_floating .text-field__input {
  width: 100%;
  /* max-width: 200px; */
  height: 100px;
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
  top: 25%;
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
