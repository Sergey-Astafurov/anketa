<template>
  <div v-if="galleryModal" class="modal-backdrop" @click="closeGallery">
    <div class="gallery-modal-content" @click.stop>
      <button class="modal-close" @click="closeGallery">×</button>
      <div class="gallery-nav">
        <button @click="prevItem" class="nav-btn">‹</button>
        <div class="gallery-display">
          <video
            v-if="galleryItems[currentIndex]?.type === 'video'"
            :src="galleryItems[currentIndex].src"
            controls
            autoplay
            class="modal-video"
          ></video>
          <img
            v-else
            :src="galleryItems[currentIndex].src"
            class="modal-image"
            alt="Фото"
          />
        </div>
        <button @click="nextItem" class="nav-btn">›</button>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  galleryModal: Boolean,
  galleryItems: Array,
  currentIndex: Number,
});
const emit = defineEmits(["close", "prev", "next"]);

function closeGallery() {
  emit("close");
}
function nextItem() {
  emit("next");
}
function prevItem() {
  emit("prev");
}
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
.gallery-modal-content {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  max-width: 90%;
  max-height: 90%;
  margin: auto;
  flex-direction: row;
  gap: 16px;
}
.gallery-nav {
  display: flex;
  align-items: center;
  gap: 12px;
}
.gallery-display {
  max-width: 800px;
  max-height: 90vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
.modal-image,
.modal-video {
  max-width: 90%;
  max-height: 90%;
  border-radius: 12px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.15);
  background: black;
}
.nav-btn {
  background: none;
  border: none;
  font-size: 48px;
  color: white;
  cursor: pointer;
  user-select: none;
  transition: transform 0.2s ease;
}
.nav-btn:hover {
  transform: scale(1.2);
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
