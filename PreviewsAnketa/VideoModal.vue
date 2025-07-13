<!-- VideoModal.vue -->
<template>
  <div v-if="visible" class="modal-backdrop" @click="close">
    <video
      class="modal-video"
      :src="src"
      controls
      autoplay
      @click.stop>
    </video>
    <button class="modal-close" @click="close">×</button>
  </div>
</template>

<script setup>
const props = defineProps({
  visible: Boolean,
  src: String
});
const emit = defineEmits(['close']);
const close = () => emit('close');
</script>

<style scoped>
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
.modal-video {
  max-width: 90%;
  max-height: 90%;
  border-radius: 12px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.15);
  background: black;
}
.modal-close {
  position: fixed;
  top: 20px;
  right: 30px;
  background: transparent;
  border: none;
  font-size: 36px;
  color: white;
  cursor: pointer;
}
</style>

