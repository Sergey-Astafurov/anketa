<template>
  <FormTitle title="Видеовизитка" />
  <div class="video-upload-form">
    <label
      v-if="!uploadedVideo"
      for="video-input"
      class="upload-area"
      tabindex="0"
      @keydown.enter.prevent="$refs.videoInput.click()"
    >
      <svg
        width="81"
        height="80"
        viewBox="0 0 81 80"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          d="M25.4999 20C25.4999 20 23.8333 26.6667 23.8333 40C23.8333 53.3333 25.4999 60 25.4999 60C25.4999 60 32.1666 58.3333 43.8333 51.6667C55.4999 45 58.8333 40 58.8333 40C58.8333 40 55.4999 35 43.8333 28.3333C32.1666 21.6667 25.4999 20 25.4999 20Z"
          stroke="#555555"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
      </svg>

      <span class="text-for-icon">Нажмите для выбора видео</span>
      <input
        ref="videoInput"
        id="video-input"
        type="file"
        accept="video/mp4,video/quicktime,video/x-msvideo"
        @change="handleVideo"
        hidden
      />
      <p class="hint">flv, avi, mov, mp4, mpeg, webm, mkv, 3gp — до 100 МБ</p>
    </label>

    <div v-if="uploadStart && !uploadStatus" class="progress">
      <progress max="100" :value="uploadPercentage"></progress>
      <span>{{ uploadPercentage }}%</span>
    </div>

    <div v-if="uploadStatus && !uploadedVideo" class="video-uploaded text-center text-white p-3">
      {{ successMessage }}
    </div>

    <div v-if="error" class="video-uploaded-error text-center text-light p-3">
      {{ errorMessage }}
    </div>

    <div v-if="uploadedVideo" class="preview-wrapper">
      <video
        class="video-preview"
        :src="uploadedVideoUrl"
        controls
        playsinline
      ></video>
      <button
        class="delete-btn"
        @click="removeVideo"
        aria-label="Удалить видео"
      >
        ×
      </button>
    </div>

    <div class="text-video">
      <div class="text-video-title">
        Видеовизитка — ваш ключ к доверию и вниманию клиентов.
      </div>
      <p class="text-video-text">
        Живое видео покажет вашу харизму и стиль, выделит вас среди других и
        поможет быстро привлечь заинтересованных.
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed } from "vue";
import axios from "axios";
import FormTitle from "@/components/ui/Forms/FormTitle.vue";

const props = defineProps({
  formData: Object,
  apiUrl: String,
  user: Object, // если нужно передавать данные пользователя для авторизации
});

const emit = defineEmits(["update", "uploadFail", "uploadVideoSuccess"]);

const videoFile = ref(null);
const uploadedVideo = ref(null);
const uploadPercentage = ref(0);
const uploadStart = ref(false);
const uploadStatus = ref(false);
const error = ref(false);
const errorMessage = ref(
  "Во время загрузки видео произошла ошибка. Попробуйте загрузить другой файл или попробуйте ещё раз."
);
const successMessage = ref(
  "Видео загружено на сервер. В данный момент мы обрабатываем его. Как только процесс завершится - видео появится в анкете."
);

const uploadedVideoUrl = computed(() => {
  if (!uploadedVideo.value) return null;
  // Если видео — строка (url) или объект с полем name, формируем url
  if (typeof uploadedVideo.value === "string") {
    return uploadedVideo.value;
  } else if (uploadedVideo.value.name) {
    return `/girls/video/${uploadedVideo.value.name}`;
  } else if (uploadedVideo.value.url) {
    return uploadedVideo.value.url;
  }
  return null;
});

function handleVideo(event) {
  const file = event.target.files[0];
  if (!file) return;

  if (file.size > 100 * 1024 * 1024) {
    alert("Файл слишком большой. Максимум 100 МБ.");
    event.target.value = "";
    return;
  }

  if (!file.type.startsWith("video/")) {
    alert("Неверный формат файла. Попробуйте загрузить другой файл.");
    event.target.value = "";
    return;
  }

  videoFile.value = file;
  uploadStart.value = true;
  uploadStatus.value = false;
  error.value = false;
  uploadPercentage.value = 0;

  const formData = new FormData();
  formData.append("video", file);

  // Подставь здесь правильный url для загрузки
  const uploadUrl = props.apiUrl + "api-girl/upload-video";

  // Если нужен авторизационный параметр
  let authParam = "";
  if (props.user && props.user.username && props.user.auth_key) {
    authParam = `&auth=${props.user.username}:${props.user.auth_key}`;
  }

  axios
    .post(uploadUrl + authParam, formData, {
      headers: { "Content-Type": "multipart/form-data" },
      onUploadProgress: (progressEvent) => {
        uploadPercentage.value = Math.round(
          (progressEvent.loaded / progressEvent.total) * 100
        );
      },
    })
    .then((res) => {
      if (!res.data.name) {
        error.value = true;
        emit("uploadFail", res);
      } else {
        uploadStatus.value = true;
        uploadedVideo.value = res.data;
        emit("uploadVideoSuccess", res.data);
        emit("update", { video: res.data });
      }
    })
    .catch((err) => {
      error.value = true;
      emit("uploadFail", err);
    });
}

function removeVideo() {
  videoFile.value = null;
  uploadStart.value = false;
  uploadStatus.value = false;
  uploadedVideo.value = null;
  error.value = false;
  uploadPercentage.value = 0;
  emit("update", { video: null });
}
</script>

<style scoped>
.text-for-icon{
  font-weight: 600;
  color: #555;
}
.video-upload-form {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 30px;
  text-align: center;
  color: #333;
}

.title {
  font-size: 1.5rem;
  margin-bottom: 12px;
  font-weight: 600;
}
.text-video {
  max-width: 360px;
}

.upload-area {
  padding: 10px;
  width: 360px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 2px dashed #c1c2c3;
  border-radius: 12px;
  height: 150px;
  cursor: pointer;
  color: #d88db1;
  background: #eef1f6;
  transition: background-color 0.3s ease;
  user-select: none;
  outline: none;
}

.upload-area:focus,
.upload-area:hover {
  background-color: #b3b3b3;
}

.upload-area .icon {
  width: 48px;
  height: 48px;
  margin-bottom: 8px;
}

.upload-area span {
  font-size: 1rem;
}

.preview-wrapper {
  position: relative;
  border-radius: 12px;
  overflow: hidden;
  width: 360px; /* Ширина около 180px */
  height: 640px; /* Высота около 320px — соотношение 9:16 */
}
@media  screen and (max-width: 480px) {
  .preview-wrapper{
    height: 350px;
  }
}

.video-preview {
  width: 100%;
  height: 100%; /* Видео растягивается на весь контейнер */
  object-fit: cover; /* Чтобы видео заполнило область по высоте и ширине без искажений */
  border-radius: 12px;
  background: black;
}

.delete-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  background: #f90042;
  border: none;
  color: white;
  font-size: 20px;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  cursor: pointer;
  line-height: 1;
  padding: 0;
  user-select: none;
  transition: background-color 0.2s ease;
}

.delete-btn:hover {
  background-color: #d60038;
}

.hint {
  max-width: 200px;
  margin-top: 8px;
  font-size: 0.85rem;
  color: #666;
}

.text-video-title {
  font-size: 20px;
  color: #555;
  margin-bottom: 10px;
  font-weight: 600;
}
.text-video-text {
  font-size: 16px;
  color: #556;
}

@media (max-width: 480px) {
  .video-upload-form {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .upload-area {
    max-width: 90%; /* почти вся ширина экрана */
    height: 140px;
  }

  .preview-wrapper {
    width: 90vw;
    height: calc(90vw * 16 / 9); /* сохранить соотношение 9:16, адаптивно */
    margin: 16px 0;
  }

  .text-video {
    max-width: 90vw;
    padding: 0 12px;
  }

  .hint {
    max-width: 100%;
  }

  .text-video-title {
    font-size: 18px;
  }

  .text-video-text {
    font-size: 14px;
  }
}
</style>
