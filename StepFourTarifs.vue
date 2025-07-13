<script setup>
import { defineProps, defineEmits } from 'vue'
import FormBase from "@/components/ui/Forms/FormBase.vue";
import FormTitle from "@/components/ui/Forms/FormTitle.vue";
import ButtonBase from "@/components/ui/ButtonBase.vue";
import ButtonOutline from "@/components/ui/ButtonOutline.vue";
import SelectTarif from '../ui/SelectTarif.vue';
const props = defineProps({
  formData: Object,
});
const emit = defineEmits(['update', 'next', 'prev'])
function updateFormData(data) {
  const updated = { ...props.formData, ...data };
  emit('update', updated);
}

function emitNext() {
  emit('update', props.formData);
  emit('next');
  window.scrollTo({ top: 0, behavior: "smooth" });
}
</script>

<template>
  <FormBase>
    <FormTitle title="Тарифы" />
    <SelectTarif :formData="formData" @update="updateFormData" />
    <div class="d-flex justify-content-between mt-4">
      <ButtonOutline @click="emit('prev')" label="Назад" />
      <ButtonBase label="Далее" @click="emitNext" />
    </div>
  </FormBase>
</template>
