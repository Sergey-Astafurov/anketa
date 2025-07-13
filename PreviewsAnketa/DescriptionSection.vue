<template>
  <div class="preview__section" v-if="description">
    <h3>Описание</h3>
    <p>{{ description }}</p>
  </div>
</template>

<script setup>
const props = defineProps({
  description: String,
});
</script>

<style scoped>
.preview__section {
  margin-top: 28px;
}
.preview__section h3 {
  font-size: 22px;
  margin-bottom: 12px;
  color: #111;
  border-bottom: 2px solid #eee;
  padding-bottom: 6px;
}
.preview__section p {
  font-size: 16px;
  line-height: 1.6;
}
</style>