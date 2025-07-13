<template>
  <FormBase>
    <PhotoUploader :formData="formData" @update="addPhotoHandler" />
    <VideoUploader :formData="formData" @update="addVideoHandler" />

    <div class="d-flex justify-content-between mt-4">
      <ButtonOutline @click="$emit('prev')" label="Назад" />
      <ButtonBase label="Далее" @click="$emit('next')" />
    </div>
  </FormBase>
</template>

<script setup>
import { toRefs, onMounted, reactive } from "vue";
import FormBase from "@/components/ui/Forms/FormBase.vue";
import PhotoUploader from "@/components/ui/Anketa/PhotoUploader.vue";
import VideoUploader from "@/components/ui/Anketa/VideoUploader.vue";
import ButtonBase from "@/components/ui/ButtonBase.vue";
import ButtonOutline from "@/components/ui/ButtonOutline.vue";
import { useRoute } from "vue-router";

const props = defineProps({
  formData: Object,
});
const emit = defineEmits(["update", "next", "prev"]);

const route = useRoute();

function addPhotoHandler(data) {
  if (!data || !Array.isArray(data.photo)) {
    console.warn("Ожидается объект с массивом photo");
    return;
  }

  // Сохраняем полностью с preview, если это черновик (ещё не опубликованная анкета)
  props.formData.photo = data.photo;

  saveAndEmit();
}


function saveAndEmit() {

  emit("update", { ...props.formData });
}

onMounted(() => {
  const savedFormData = localStorage.getItem('formData');
  if (savedFormData) {
    try {
      const data = JSON.parse(savedFormData);
      // Обновляем свойства объекта props.formData без замены объекта
      Object.keys(data).forEach(key => {
        props.formData[key] = data[key];
      });
      emit("update", { ...props.formData });
    } catch (e) {
      console.error('Ошибка парсинга formData из localStorage', e);
    }
  }
});

</script>
