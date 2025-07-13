<template>
  <div class="preview__top-layout">
    <!-- Инфо для мобилок -->
    <div v-if="isMobile">
      <h2>{{ formData.name }}, {{ formData.age }} лет</h2>
      <p class="tags-pills mb-3">
        <a class="pill">{{ formData.city }}</a>
        <a v-if="formData.district" class="pill">{{ formData.district }}</a>
        <a v-if="formData.metro" class="pill">{{ formData.metro }}</a>
      </p>
    </div>

    <!-- Swiper: основная галерея -->
    <div class="preview__gallery">
      <Swiper
        class="main-swiper"
        :modules="[Navigation, Thumbs]"
        :thumbs="{ swiper: thumbsSwiper }"
        navigation
        space-between="10"
        slides-per-view="1">
        <!-- Главное фото -->
        <SwiperSlide>
          <img
            :src="mainPhoto"
            class="main-photo-slide"
            @click="$emit('open-preview', mainPhoto)"
            alt="Главное фото" />
        </SwiperSlide>

        <!-- Видеовизитка -->
        <SwiperSlide v-if="videoUrl">
          <video controls class="main-video-slide">
            <source :src="videoUrl" type="video/mp4" />
            Ваш браузер не поддерживает видео.
          </video>
        </SwiperSlide>

        <!-- Остальные фото -->
        <SwiperSlide
          v-for="(photo, index) in otherPhotos"
          :key="'slide-' + index">
          <img
            :src="photo"
            class="main-photo-slide"
            @click="$emit('open-preview', photo)"
            alt="Фото" />
        </SwiperSlide>
      </Swiper>
      <Swiper
        class="thumbs-swiper mt-3"
        :modules="[FreeMode, Navigation, Thumbs]"
        space-between="10"
        slides-per-view="auto"
        free-mode
        watch-slides-progress
        @swiper="setThumbsSwiper">
        <!-- thumb mainPhoto -->
        <SwiperSlide class="thumb-slide" :key="'thumb-main'">
          <img :src="mainPhoto" alt="Thumb main" />
        </SwiperSlide>

        <!-- thumb video -->
        <SwiperSlide
          v-if="formData.videoUrl"
          class="thumb-slide"
          :key="'thumb-video'">
          <video muted class="thumb-video" :src="formData.videoUrl" />
        </SwiperSlide>

        <!-- thumb остальные фото -->
        <SwiperSlide
          v-for="(photo, index) in otherPhotos"
          :key="'thumb-' + index"
          class="thumb-slide">
          <img :src="photo" alt="Thumb photo" />
        </SwiperSlide>
      </Swiper>
    </div>
    <!-- Вся остальная инфа -->
    <div class="preview__info">
      <div class="is-mob">
        <h2>{{ formData.name }}, {{ formData.age }} лет</h2>
        <p class="tags-pills mb-3">
          <a class="pill">{{ formData.city }}</a>
          <a v-if="formData.district" class="pill">{{ formData.district }}</a>
          <a v-if="formData.metro" class="pill">{{ formData.metro }}</a>
        </p>
      </div>

      <!-- Блок с описанием -->
      <div class="container bg-light rounded p-3 mb-4 shadow-sm">
        <div class="row gy-3">
          <div class="col-md-4" v-if="formData.height || formData.weight">
            <strong>Рост / Вес / Грудь:</strong>
            <div class="desc">
              {{ formData.height }} / {{ formData.weight }} /
              {{ breastLabel }}
            </div>
          </div>

          <div class="col-md-4" v-if="figureLabel">
            <strong>Фигура:</strong>
            <div class="desc">{{ figureLabel }}</div>
          </div>

          <div class="col-md-4" v-if="hairColor">
            <strong>Цвет волос:</strong>
            <div class="desc">{{ hairColor }}</div>
          </div>

          <div class="col-md-4" v-if="formData.nationality">
            <strong>Национальность:</strong>
            <div class="desc">{{ formData.nationality }}</div>
          </div>

          <div class="col-md-4" v-if="formData.appearance">
            <strong>Внешность:</strong>
            <div class="desc">{{ formData.appearance }}</div>
          </div>

          <div class="col-md-4" v-if="formData.intim_haircut">
            <strong>Интимная стрижка:</strong>
            <div class="desc">{{ formData.intim_haircut }}</div>
          </div>

          <div class="col-md-4" v-if="clothingSizeLabel || formData.shoe_size">
            <strong>Одежда / Обувь:</strong>
            <div class="desc">
              {{ clothingSizeLabel }} ⁄ {{ formData.shoe_size }}
            </div>
          </div>

          <div class="col-md-4" v-if="formData.not_younger">
            <strong>Возраст клиента:</strong>
            <div class="desc">
              {{ formData.not_younger }} – {{ formData.not_older }}
            </div>
          </div>
        </div>
      </div>

      <!-- Теги -->
      <div v-if="formData.tags?.length" class="tags-pills">
        <span v-for="tag in formData.tags" :key="tag" class="pill">{{
          tag
        }}</span>
      </div>

      <!-- Контакты -->
      <div class="contacts-buttons">
        <button
          v-if="formData.tel && !showPhone"
          class="contact-btn phone"
          @click="showPhone = true">
          Показать номер
        </button>

        <a
          v-if="formData.tel && showPhone"
          :href="`tel:${formData.tel}`"
          class="contact-btn phone"
          target="_blank">
          {{ formData.tel }}
        </a>

        <a
          v-if="formData.whatsapp"
          :href="`https://wa.me/${formatPhoneForWhatsapp(formData.whatsapp)}`"
          class="contact-btn whatsapp"
          target="_blank">
          WhatsApp
        </a>

        <a
          v-if="formData.tg"
          :href="`https://t.me/${formData.tg.replace(/^@/, '')}`"
          class="contact-btn telegram"
          target="_blank">
          Telegram
        </a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import { Navigation, Thumbs, FreeMode } from "swiper/modules";

// Стили Swiper
import "swiper/css";
import "swiper/css/navigation";
import "swiper/css/thumbs";
import "swiper/css/free-mode";

const props = defineProps({
  formData: Object,
  mainPhoto: String,
  figureLabel: String,
  hairColor: String,
  tagPills: Array,
  isMobile: Boolean,
  clothingSizeLabel: String,
  breastLabel: String,
});

const emit = defineEmits(["open-preview"]);

const showPhone = ref(false);
const thumbsSwiper = ref(null);
const setThumbsSwiper = (swiper) => {
  thumbsSwiper.value = swiper;
};

const videoUrl = computed(() => {
  const video = props.formData.video;
  return video instanceof File ? URL.createObjectURL(video) : video || "";
});

function formatPhoneForWhatsapp(phone) {
  return typeof phone === "string" ? phone.replace(/\D/g, "") : "";
}

const photoPreviews = computed(() => {
  const photos = props.formData.photo || [];

  return photos.map((p) => {
    if (typeof p === 'string') {
      return { file: null, url: p };
    }

    if (p?.preview) {
      return { file: null, url: p.preview };
    }

    if (p instanceof File) {
      return { file: p, url: URL.createObjectURL(p) };
    }

    return { file: null, url: '' };
  });
});

const mainPhoto = computed(() => {
  return photoPreviews.value.length ? photoPreviews.value[0].url : "";
});


// Получаем остальные фото, исключая главное
const otherPhotos = computed(() => {
  return photoPreviews.value
    .filter((p) => p.url !== mainPhoto.value)
    .map((p) => p.url);
});



</script>

<style scoped>
.main-photo-slide {
  width: 100%;
  height: 600px;
  border-radius: 10px;
  object-fit: cover;
  cursor: pointer;
}
.main-video-slide {
  width: 100%;
  height: 600px;
  border-radius: 10px;
  object-fit: cover;
}
.thumbs-swiper {
  max-width: 100%;
  height: 80px;
}
.thumb-slide {
  width: 80px !important;
  height: 80px;
  cursor: pointer;
  border-radius: 6px;
  overflow: hidden;
}
.thumb-slide img,
.thumb-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.thumb-video {
  pointer-events: none;
}
@media screen and (max-width: 768px) {
  .is-mob {
    display: none;
  }
}
.desc {
  color: #555;
  font-weight: 600;
}
.preview__top-layout {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.preview__gallery {
  max-width: 400px;
  flex: 1 1 60%;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.preview__info {
  flex: 1 1 35%;
}

.main-photo-slide,
.main-video-slide {
  width: 100%;
  height: 600px;
  border-radius: 10px;
  overflow: hidden;
  object-fit: cover;
  cursor: pointer;
}

.thumbs-swiper {
  height: 80px;
}

.thumb-slide {
  width: 80px !important;
  height: 80px;
  border-radius: 6px;
  overflow: hidden;
  cursor: pointer;
}

.thumb-slide img,
.thumb-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.main-swiper {
  height: 600px;
}
@media screen and (max-width: 768px) {
  .preview__top-layout {
    flex-direction: column;
  }

  .preview__gallery,
  .preview__info {
    flex: 1 1 100%;
  }

  .is-mob {
    display: block;
  }
}
</style>
