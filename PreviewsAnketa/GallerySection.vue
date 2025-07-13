<template>
  <div v-if="photos.length > 0 || videoUrl" class="gallery">
    <h3>Галерея</h3>
    <div class="gallery__list">
      <div
        v-for="(item, i) in galleryItems"
        :key="i"
        class="gallery__thumb"
        @click="$emit('open-preview', i)"
      >
        <template v-if="item.type === 'video'">
          <video :src="item.src" muted preload="metadata" @click="$emit('open-preview', 0)" class="gallery-thumb" />
          <div class="play-icon">▶</div>
        </template>
        <template v-else>
          <img :src="item.src" class="gallery-thumb" alt="Фото" />
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  photos: Array,
  videoUrl: String,
  galleryItems: Array
})
const emit = defineEmits(["open-preview"]);
</script>

<style scoped>
.gallery {
  margin-bottom: 32px;
}
.gallery__list {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 8px;
}
.gallery__thumb {
  position: relative;
  width: 100px;
  height: 130px;
  border-radius: 10px;
  overflow: hidden;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}
.gallery-thumb {
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: brightness(0.8);
}
.play-icon {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 28px;
  pointer-events: none;
  user-select: none;
}
</style>