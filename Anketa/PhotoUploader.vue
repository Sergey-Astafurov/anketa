<script setup>
import { ref, onMounted, watch, computed } from "vue";
import axios from "axios";
import FormTitle from "@/components/ui/Forms/FormTitle.vue";
import { useStore } from "vuex";
const store = useStore();

const props = defineProps({
  formData: Object,
});

const emit = defineEmits(["update"]);

const user = computed(() => store.getters.getUser);
const apiUrl = computed(() => store.getters.getApiUrl);
const apiDomine = computed(() => store.getters.getApiDomine);
const uploadLink = computed(() => `${apiUrl.value}api-girl/upload-photo`);
const photos = ref([]); // { name, preview }
const notification = ref("");

function showNotification(msg) {
  notification.value = msg;
  setTimeout(() => (notification.value = ""), 3000);
}

function handleFiles(event) {
  const files = Array.from(event.target.files);
  const availableSlots = 10 - photos.value.length;

  if (availableSlots <= 0) {
    showNotification("Максимум 10 фотографий");
    return;
  }

  const filesToUpload = files.slice(0, availableSlots);
  filesToUpload.forEach((file) => {
    if (!["image/jpeg", "image/png", "image/webp"].includes(file.type)) {
      showNotification("Недопустимый формат файла: " + file.type);
      return;
    }
    uploadFile(file);
  });

  event.target.value = "";
}

async function uploadFile(file) {
  try {
    const formData = new FormData();
    formData.append("image", file);
    console.log(user.value.username);
    console.log(uploadLink.value);
    console.log(user.value.auth_key);
   const url = `${uploadLink.value}&auth=${user.value.username}:${user.value.auth_key}`;

    const { data } = await axios.post(
      url,
      formData,
      {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      }
    );

    if (!data.name) {
      showNotification("Ошибка загрузки фото");
      return;
    }

    photos.value.push({
      name: `${data.name}`, // <-- добавляем относительный путь
      preview: URL.createObjectURL(file),
    });

    emitPhotoUpdate();
  } catch (err) {
    showNotification("Ошибка загрузки фото");
    console.error(err);
  }
}

function removePhoto(index) {
  URL.revokeObjectURL(photos.value[index].preview);
  photos.value.splice(index, 1);
  emitPhotoUpdate();
}

function makeMain(index) {
  const [selected] = photos.value.splice(index, 1);
  photos.value.unshift(selected);
  emitPhotoUpdate();
}

function emitPhotoUpdate() {
  emit("update", {
    photo: photos.value.map((p) => ({
      name: p.name,
      preview: p.preview,
    })),
  });
}
onMounted(() => {
  if (Array.isArray(props.formData.photo)) {
    photos.value = props.formData.photo.map((p) => {
      const cleanName = p.name;
      return {
        name: cleanName,
        preview: `${apiDomine}${cleanName}`,
      };
    });
  }
});
</script>

<template>
  <div>
    <FormTitle title="Фото" />
    <p style="color: #555; margin-bottom: 20px">
      Перенесите фото на первое место, чтобы сделать его главным. Фото до 25Mb
      формата png, jpeg, jpg, webp. Лимит — 10 штук.
    </p>

    <div class="preview-wrapper mb-3">
      <div class="preview-items">
        <TransitionGroup
          name="fade-slide"
          tag="div"
          class="preview-items-inner">
          <label class="upload-box" for="photo-input" :key="'upload-box'"
            >+
            <input
              id="photo-input"
              type="file"
              accept="image/png,image/jpeg,image/webp"
              multiple
              @change="handleFiles"
              hidden />
          </label>

          <div
            v-for="(file, index) in photos"
            :key="file.id"
            class="preview-item">
            <img :src="file.preview" alt="Фото" />
            <button class="delete-btn" @click="removePhoto(index)">×</button>
            <button
              v-if="index !== 0"
              class="main-btn"
              @click="makeMain(index)"
              title="Сделать главным">
              ★
            </button>
            <div v-if="index === 0" class="main-label">Главное фото</div>
          </div>
        </TransitionGroup>
      </div>
    </div>

    <div v-if="notification" class="notification">{{ notification }}</div>
  </div>
</template>

<style scoped>
.upload-section {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  flex-wrap: wrap;
}
.preview-items {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}
.preview-items-inner {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}
.upload-box {
  width: 120px;
  height: 120px;
  border: 2px dashed #c1c2c3;
  border-radius: 12px;
  font-size: 48px;
  color: #555;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  background-color: #eef1f6;
  transition: background-color 0.3s ease;
}

.upload-box:hover {
  background-color: #b3b3b3;
}

.preview-wrapper {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.preview-item {
  position: relative;
  width: 120px;
  height: 120px;
}

.preview-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 12px;
}

.delete-btn {
  position: absolute;
  top: -6px;
  right: -6px;
  background: #f90042;
  border: none;
  color: white;
  border-radius: 50%;
  font-size: 18px;
  width: 24px;
  height: 24px;
  cursor: pointer;
  line-height: 1;
}

.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.3s ease;
}

.fade-slide-enter-from,
.fade-slide-leave-to {
  opacity: 0;
  transform: scale(0.9) translateY(10px);
}
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  animation: fadeIn 0.3s ease;
}

.modal-image {
  max-width: 90%;
  max-height: 90%;
  border-radius: 8px;
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.2);
}

.modal-close {
  position: fixed;
  top: 20px;
  right: 30px;
  background: transparent;
  color: white;
  border: none;
  font-size: 36px;
  cursor: pointer;
  z-index: 10000;
  line-height: 1;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.97);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.main-btn {
  position: absolute;
  bottom: 6px;
  right: 6px;
  background: #ffd700;
  border: none;
  border-radius: 50%;
  color: white;
  font-size: 18px;
  width: 24px;
  height: 24px;
  cursor: pointer;
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.3);
  transition: transform 0.2s ease;
}
.main-btn:hover {
  transform: scale(1.1);
}

.main-label {
  position: absolute;
  bottom: 6px;
  left: 6px;
  background: rgba(0, 0, 0, 0.6);
  color: white;
  font-size: 12px;
  padding: 2px 6px;
  border-radius: 6px;
}
</style>
