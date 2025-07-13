<template>
  <div class="preview">
    <PreviewTop
      :formData="formData"
      :is-mobile="isMobile"
      :mainPhoto="mainPhoto"
      :showPhone="showPhone"
      @togglePhone="showPhone = !showPhone"
      :hair-color="hairColor"
      :clothing-size-label="clothingSizeLabel"
      @open-preview="openPreview"
      :breast-label="breastLabel"
      :figure-label="figureLabel" />

    <DescriptionSection :description="formData.description" />
    <PreferencesSection :preferences="preparedPreferences" />
    <TariffSection :tariffs="formData.tariffs" />

    <div class="d-flex justify-content-between mt-4">
      <ButtonOutline @click="$emit('prev')" label="Назад" />
     <ButtonBase label="Опубликовать" @click="() => emit('submit')" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";

import ButtonBase from "@/components/ui/ButtonBase.vue";
import ButtonOutline from "@/components/ui/ButtonOutline.vue";
import PreviewTop from "@/components/blocks/PreviewsAnketa/PreviewTop.vue";
import GallerySection from "@/components/blocks/PreviewsAnketa/GallerySection.vue";
import DescriptionSection from "@/components/blocks/PreviewsAnketa/DescriptionSection.vue";
import PreferencesSection from "@/components/blocks/PreviewsAnketa/PreferencesSection.vue";
import TariffSection from "@/components/blocks/PreviewsAnketa/TariffsSection.vue";
import GalleryModals from "@/components/blocks/PreviewsAnketa/GalleryModal.vue";

const props = defineProps({
  formData: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(["submit", "prev"]);

const fullPreview = ref(null);
const galleryModal = ref(false);
const videoModal = ref(false);

const currentIndex = ref(0);
const showPhone = ref(false);

const selectedPreferences = ref([]);
const preferenceComments = ref({});
const tagPills = ref([]);

const isMobile = ref(window.innerWidth < 768);

// Реактивные вычисления меток
const figureLabel = computed(() => {
  switch (props.formData.body_type) {
    case "slim":
      return "Стройная";
    case "medium":
      return "Средняя";
    case "curvy":
      return "Пышная";
    case "athletic":
      return "Спортивная";
    default:
      return "";
  }
});

const clothingSizeLabel = computed(() => {
  switch (props.formData.clothing_size) {
    case "1":
      return "XS";
    case "2":
      return "S";
    case "3":
      return "M";
    case "4":
      return "L";
    case "5":
      return "XL";
    case "6":
      return "больше XL";
    default:
      return "";
  }
});

const hairColor = computed(() => {
  switch (props.formData.hairColor) {
    case "blond":
      return "Блондинка";
    case "black":
      return "Брюнетка";
    case "brown":
      return "Шатенка";
    case "red":
      return "Рыжая";
    case "dyed":
      return "Крашеная";
    default:
      return "";
  }
});

const breastLabel = computed(() => {
  switch (props.formData.breast) {
    case "1":
      return "Первый размер";
    case "2":
      return "Второй размер";
    case "3":
      return "Третий размер";
    case "4":
      return "Четвертый размер";
    case "5":
      return "Пятый размер";
    default:
      return "";
  }
});

// Преобразуем фото из formData
const photoPreviews = computed(() => {
  const photos = props.formData.photo || [];

  return photos.map((p) => {
    let url = "";

    if (typeof p === "string") {
      url = p;
    } else if (p?.preview) {
      url = p.preview;
    } else if (p?.name) {
      const cleanName = p.name
      url = `${apiDomine}${cleanName}`;
    }

    return {
      file: null,
      url,
    };
  });
});


// Основное фото — первый из photoPreviews

const mainPhoto = computed(() => {
  return photoPreviews.value.length ? photoPreviews.value[0].url : "";
});
// Фото из formData (если не File, а URL)
const photos = computed(() => {
  return props.formData.photo || [];
});

// Видео
const videoUrl = computed(() => {
  const video = props.formData.video;
  return video instanceof File ? URL.createObjectURL(video) : video || "";
});

// Подготовка тегов с комментариями
const preparedPreferences = computed(() => {
  const service = props.formData.service || {};
  const result = [];

  for (const category in service) {
    if (Array.isArray(service[category])) {
      service[category].forEach(item => {
        if (item.name) {
          result.push({
            key: item.name,
            label: item.name,
            comment: item.description || null,
          });
        }
      });
    }
  }

  return result;
});

// Галерея: сначала видео, потом фото
const galleryItems = computed(() => {
  const items = [];
  if (videoUrl.value) items.push({ type: "video", src: videoUrl.value });
  photoPreviews.value.forEach((p) => {
    items.push({ type: "image", src: p.url });
  });
  return items;
});


// Обработчики
function openPreview(src) {
  fullPreview.value = src;
}
function closePreview() {
  fullPreview.value = null;
}

function openGalleryAt(index) {
  currentIndex.value = index;
  galleryModal.value = true;
}
function closeGallery() {
  galleryModal.value = false;
}

function nextItem() {
  currentIndex.value = (currentIndex.value + 1) % galleryItems.value.length;
}
function prevItem() {
  currentIndex.value =
    (currentIndex.value - 1 + galleryItems.value.length) % galleryItems.value.length;
}

function handleKeydown(e) {
  if (!galleryModal.value) return;
  if (e.key === "Escape") closeGallery();
  if (e.key === "ArrowRight") nextItem();
  if (e.key === "ArrowLeft") prevItem();
}

function checkWidth() {
  isMobile.value = window.innerWidth < 768;
}

// Загрузка из localStorage один раз при монтировании
onMounted(() => {
  window.addEventListener("resize", checkWidth);
  window.addEventListener("keydown", handleKeydown);

  checkWidth();

  try {
    const saved = localStorage.getItem("anketa-form-data");
    if (!saved) return;
    const parsed = JSON.parse(saved);
    selectedPreferences.value = parsed.selectedSubcategories || [];
    preferenceComments.value = parsed.preferenceComments || {};
    tagPills.value = parsed.preferences || [];
  } catch (err) {
    console.error("Ошибка при загрузке анкеты из localStorage", err);
    selectedPreferences.value = [];
    preferenceComments.value = {};
    tagPills.value = [];
  }
});

// Очистка URL.createObjectURL при размонтировании
onUnmounted(() => {
  window.removeEventListener("resize", checkWidth);
  window.removeEventListener("keydown", handleKeydown);

  photoPreviews.value.forEach((p) => URL.revokeObjectURL(p.url));
  if (videoUrl.value) URL.revokeObjectURL(videoUrl.value);
});
</script>


<style>
.preview {
  max-width: 960px;
  padding: 32px;
  margin: 0 auto;
  background: #fff;
  border-radius: 24px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  animation: fadeIn 0.4s ease-out;
  font-family: "Segoe UI", sans-serif;
  color: #222;
}

.char-list {
  display: flex;
  gap: 5px;
  color: #555;
  font-weight: 600;
}
.preview__top {
  display: flex;
  /* flex-wrap: wrap; */
  gap: 32px;
  margin-bottom: 32px;
}
@media (max-width: 768px) {
  .preview__top {
    flex-wrap: wrap;
  }
}

.main-photo {
  width: 260px;
  height: 360px;
  object-fit: cover;
  border-radius: 16px;
  box-shadow: 0 4px 14px rgba(0, 0, 0, 0.15);
  cursor: pointer;
  transition: transform 0.3s ease;
}
.main-photo:hover {
  transform: scale(1.03);
}

.preview__info h2 {
  font-size: 30px;
  margin-bottom: 12px;
}
.preview__info p {
  margin: 4px 0;
  font-size: 16px;
  color: #444;
}

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

/* Галерея */
.gallery {
  margin-bottom: 32px;
}
.gallery__list {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 8px;
}
.gallery__item {
  width: 100px;
  height: 130px;
  object-fit: cover;
  border-radius: 10px;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease;
}
.gallery__item:hover {
  transform: scale(1.04);
}

.tags {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
}
/* Тарифы */
.tariff-cards.sexy {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-top: 14px;
  justify-content: flex-start;
}

.tariff-card.sexy {
  flex: 1 1 200px;
  background: linear-gradient(145deg, #3d2c41, #271c2e);
  border: 1px solid #5e3b6b;
  border-radius: 18px;
  padding: 18px;
  color: #f3d9ff;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.35);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  font-family: "Segoe UI", sans-serif;
  backdrop-filter: blur(6px);
}

.tariff-card.sexy.day {
  background: linear-gradient(135deg, #4d2c5c, #2e153d);
}

.tariff-card.sexy.night {
  background: linear-gradient(135deg, #1c1b29, #2a0e3f);
}

.tariff-card.sexy:hover {
  transform: translateY(-4px) scale(1.02);
  box-shadow: 0 12px 28px rgba(0, 0, 0, 0.45);
}

.tariff-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 16px;
  margin-bottom: 10px;
  font-weight: 600;
}

.tariff-time {
  font-size: 18px;
  font-weight: 700;
}

.tariff-icon {
  font-size: 20px;
}

.tariff-price p {
  margin: 4px 0;
  font-size: 15px;
}

.tariff-price span {
  font-weight: bold;
  color: #f9a8d4;
}
/* Услуги */
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

/* Видео */
/* video {
  max-width: 100%;
  border-radius: 12px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.15);
} */

/* Превью-модалка */
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
.modal-image {
  max-width: 90%;
  max-height: 90%;
  border-radius: 12px;
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
.tariff-group {
  margin-top: 12px;
}

.tariff-label {
  font-size: 18px;
  font-weight: bold;
  margin: 16px 0 8px;
  color: #333;
  border-left: 4px solid #ccc;
  padding-left: 10px;
}

.tariff-card {
  border-radius: 12px;
  padding: 14px 18px;
  min-width: 140px;
  flex: 1;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.08);
  transition: transform 0.2s ease;
}

.tariff-card.day {
  background: linear-gradient(135deg, #e8f8ff, #d4f1ff);
  border: 1px solid #aee1f9;
}

.tariff-card.night {
  background: linear-gradient(135deg, #eee4ff, #d6caff);
  border: 1px solid #b4a0f0;
}

.tariff-card:hover {
  transform: translateY(-3px);
}

.tariff-card .time {
  font-weight: bold;
  font-size: 16px;
  margin-bottom: 6px;
}

.price {
  font-weight: bold;
  font-size: 20px;
  position: relative;
}

.price::after {
  content: "₽";
  font-size: 14px;
  position: absolute;
  top: 0;
  right: -16px;
  color: #1455a3;
}

.contacts-buttons {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  margin-top: 12px;
}

.contact-btn {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 10px 18px;
  font-size: 15px;
  font-weight: 500;
  border: 1px solid rgba(0, 0, 0, 0.06);
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.6);
  color: #222;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.06);
  backdrop-filter: blur(6px);
}

.contact-btn i {
  font-size: 16px;
  color: #555;
}

/* Анимация */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(12px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
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

.gallery__video-preview {
  position: relative;
  width: 100px;
  height: 130px;
  cursor: pointer;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-right: 12px;
  flex-shrink: 0;
}

.gallery__video-thumb {
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: brightness(0.7);
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

.modal-video {
  max-width: 90%;
  max-height: 90%;
  border-radius: 12px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.15);
  background: black;
}

.tags-pills {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 12px;
}

.pill {
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 13px;
  color: #333;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
  transition: all 0.25s ease;
  cursor: pointer;
}

.pill:hover {
  background-color: #ffeef2;
  border-color: #f35c7a;
  color: #d12244;
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
</style>
