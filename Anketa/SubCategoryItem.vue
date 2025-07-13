<template>
  <div
    class="subcategory-item"
    :class="{ active: isSelected }"
    @click="toggle"
  >
    <div class="subcategory-label">{{ subcategory }}</div>
    <input
      v-if="isSelected"
      type="text"
      v-model="comment"
      @click.stop
      placeholder="Комментарий"
      class="comment-input"
    />
  </div>
</template>

<script setup>
import { watch, ref, toRefs } from "vue";

const props = defineProps({
  subcategory: String,
  modelValue: Boolean,
  commentValue: String,
});
const emit = defineEmits(["update:modelValue", "update:commentValue"]);

const isSelected = ref(props.modelValue ?? false);
const comment = ref(props.commentValue ?? "");

watch(() => props.modelValue, (v) => {
  isSelected.value = v;
});
watch(() => props.commentValue, (v) => {
  comment.value = v;
});

function toggle() {
  isSelected.value = !isSelected.value;
  emit("update:modelValue", isSelected.value);
  if (!isSelected.value) {
    comment.value = "";
    emit("update:commentValue", "");
  }
}
watch(comment, (val) => {
  emit("update:commentValue", val);
});
</script>

<style scoped>
.subcategory-item {
  display: inline-flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 8px 14px;
  border-radius: 20px;
  border: 1.5px solid #ccc;
  background-color: #fff;
  color: #333;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 15px;
  max-width: 100%;
  position: relative;
  margin: 6px;
}
.subcategory-item:hover {
  border-color: #f90042;
}
.subcategory-item.active {
  border-color: #f90042;
  background-color: #fff0f5;
  color: #f90042;
  box-shadow: 0 2px 6px rgba(249, 0, 66, 0.1);
}
.comment-input {
  margin-top: 6px;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 6px 10px;
  font-size: 14px;
  background: #fff;
  width: 100%;
  transition: border-color 0.2s ease;
}
.comment-input:focus {
  border-color: #f90042;
}
.subcategory-label {
  pointer-events: none; /* чтобы текст не мешал клику */
}
</style>
