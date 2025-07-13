<script setup>
import { computed, ref, onMounted, onBeforeUnmount } from "vue";

const props = defineProps({
  options: { type: Array, required: true },
  modelValue: [String, Number, null],
  placeholder: { type: String, default: "Выберите значение" },
  openId: { type: Object, required: true },
  selectId: { type: String, required: true },
  label: { type: String, default: "" },
});

const emit = defineEmits(["update:modelValue"]);

const isFocused = ref(false);

const isOpen = computed(() => props.openId.value === props.selectId);

const selectedOption = computed(() =>
  props.options.find(
    (opt) => opt.value === props.modelValue || opt.id === props.modelValue
  )
);

const selectedLabel = computed(() => selectedOption.value?.label || "");

function toggleDropdown() {
  if (props.openId.value === props.selectId) {
    props.openId.value = null;
  } else {
    props.openId.value = props.selectId;
  }
  isFocused.value = false;
}

function selectOption(option) {
  emit("update:modelValue", option.value ?? option.id);
  props.openId.value = null;
  isFocused.value = false;
}

function handleClickOutside(e) {
  const el = document.querySelector(`[data-id="${props.selectId}"]`);
  if (el && !el.contains(e.target)) {
    props.openId.value = null;
    isFocused.value = false;
  }
}

onMounted(() => document.addEventListener("click", handleClickOutside));
onBeforeUnmount(() =>
  document.removeEventListener("click", handleClickOutside)
);
</script>

<template>
  <div class="floating-select-wrapper" :data-id="selectId">
    <label
      class="floating-label"
      :class="{ floated: isFocused || selectedOption }">
      {{ label }}
    </label>

    <div class="custom-select" @click.stop="toggleDropdown">
      <div class="selected">
        <span :class="{ placeholder: !selectedOption }">
          {{ selectedLabel || placeholder }}
        </span>
      </div>
      <span class="arrow" :class="{ open: isOpen }"><i class="bi bi-chevron-down"></i></span>

      <transition name="fade">
        <ul v-if="isOpen" class="options">
          <li
            v-for="option in options"
            :key="option.value ?? option.id"
            @click.stop="selectOption(option)">
            {{ option.label }}
          </li>
        </ul>
      </transition>
    </div>
  </div>
</template>

<style scoped>
.floating-select-wrapper {
  position: relative;
  width: 100%;
}

.floating-label {
  position: absolute;
  top: 50%;
  left: 0.75rem;
  transform: translateY(-50%);
  padding: 0 0.25rem;
  font-size: 1rem;
  pointer-events: none;
  transition: transform 0.15s ease-in-out, font-size 0.15s ease-in-out,
    opacity 0.15s ease-in-out;
  z-index: 100;
}

.floating-label.floated {
  transform: translateY(-1.5rem) scale(0.85);
  font-size: 0.875rem;
  opacity: 0.65;
  background: transparent;
  padding: 0 0.25rem;
}

.custom-select:focus,
.custom-select:not(:placeholder-shown) {
  padding-top: 1.625rem;
  padding-bottom: 0.625rem;
}

 .custom-select:focus ~ .floating-label,
  .custom-selec:not(:placeholder-shown)
  ~ .floating-label{
  transform: translateY(-1.5rem) scale(0.85);
  font-size: 0.875rem;
  opacity: 0.65;
  background: transparent;
  padding: 0 0.25rem;
}

.custom-select {
  width: 100%;
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
.custom-select:hover {
  border-color: #f90042;
}
.selected {
  position: relative;
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  /* z-index: 10000; */
}
.placeholder {
  color: #bbb;
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
.arrow.open {
  transform: rotate(180deg);
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
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
