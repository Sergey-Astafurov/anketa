<template>
  <div class="preview__section" v-if="preferences.length">
    <h3>Услуги</h3>
    <transition-group name="tag-fade" tag="div" class="tags">
      <div
        v-for="pref in preferences"
        :key="pref.key"
        class="tag tag--full">
        <span>{{ pref.label }}</span>
        <span v-if="pref.comment" class="tag-comment">— {{ pref.comment }}</span>
      </div>
    </transition-group>
  </div>
</template>

<script setup>
const props = defineProps({
  preferences: Array,
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
.tags {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
}
.tag--full {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border-radius: 20px;
  background: #e0ecff;
  color: #1455a3;
  font-size: 14px;
  line-height: 1.3;
}
.tag-comment {
  font-style: italic;
  color: #555;
  font-size: 13px;
}
.tag-fade-enter-active,
.tag-fade-leave-active {
  transition: all 0.4s ease;
}
.tag-fade-enter-from,
.tag-fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
.tag-fade-enter-to,
.tag-fade-leave-from {
  opacity: 1;
  transform: translateY(0);
}
</style>
