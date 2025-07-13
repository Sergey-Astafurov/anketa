<template>
  <FormBase>
    <FormTitle title="Параметры" />
    <div class="params mb-5">
      <div class="">
        <NumberWithSuffix
          v-model="localForm.height"
          suffix="см"
          label="Рост"
          name="height" />
      </div>
      <div class="">
        <NumberWithSuffix
          v-model="localForm.weight"
          suffix="кг"
          label="Вес"
          name="weight" />
      </div>
      <div class="">
        <TheSelect
          v-model="localForm.age"
          :options="ages"
          :open-id="sharedOpenId"
          select-id="age"
          label="Возраст"
          placeholder=" " />
      </div>
      <div class="">
        <TheSelect
          v-model="localForm.breast"
          :options="selectBreast"
          :open-id="sharedOpenId"
          label="Размер груди"
          select-id="breast"
          placeholder=" " />
      </div>
      <div class="">
        <TheSelect
          v-model="localForm.clothing_size"
          :options="selectClothingSize"
          :open-id="sharedOpenId"
          label="Размер одежды"
          select-id="сlothing"
          placeholder=" " />
      </div>
      <div class="">
        <TheSelect
          v-model="localForm.shoe_size"
          :options="selectShoe"
          :open-id="sharedOpenId"
          label="Размер обуви"
          select-id="shoe"
          placeholder=" " />
      </div>
    </div>
    <FormTitle title="Внешность" />
    <div class="params mb-5">
      <div class="">
        <TheSelect
          v-model="localForm.nationality"
          :options="selectNationality"
          :open-id="sharedOpenId"
          label="Национальность"
          select-id="nationality"
          placeholder=" " />
      </div>
      <div class="">
        <TheSelect
          v-model="localForm.appearance"
          :options="selectAppearance"
          :open-id="sharedOpenId"
          label="Внешность"
          select-id="appearance"
          placeholder=" " />
      </div>

      <div class="">
        <TheSelect
          v-model="localForm.intim_haircut"
          :options="selectIntimHaircut"
          :open-id="sharedOpenId"
          select-id="haircut"
          label="Интимная стрижка"
          placeholder=" " />
      </div>
      <SelectColor
        v-model:selected="localForm.hairColor"
        :hair-colors="hairColors" />
      <TheRadioInput v-model="localForm.body_type" :figures="figures" />
    </div>
    <FormTitle title="Дополнительно" />
    <div class="mb-5">
      <RangeInput v-model:ageRange="localAgeRange" />
    </div>
    <div class="d-flex justify-content-between mt-4">
      <ButtonOutline @click="$emit('prev')" label="Назад" />
      <ButtonBase label="Далее" @click="emitNext" />
    </div>
  </FormBase>
</template>

<script setup>
import FormBase from "@/components/ui/Forms/FormBase.vue";
import NumberWithSuffix from "@/components/ui/Forms/NumberWithSuffix.vue";
import FormTitle from "@/components/ui/Forms/FormTitle.vue";
import ButtonBase from "@/components/ui/ButtonBase.vue";
import ButtonOutline from "@/components/ui/ButtonOutline.vue";
import TheSelect from "@/components/ui/Forms/TheSelect.vue";
import SelectColor from "@/components/ui/Forms/SelectColor.vue";
import RangeInput from "@/components/ui/Forms/RangeInput.vue";
import TheRadioInput from "@/components/ui/Forms/TheRadioInput.vue";
import { reactive, watch, ref, toRef, computed } from "vue";

const props = defineProps({
  formData: Object,
});
const openSelectId = ref(null);
const sharedOpenId = toRef({ value: openSelectId });
const ages = Array.from({ length: 48 }, (_, i) => ({
  label: (i + 18).toString(),
  value: i + 18,
}));
const selectShoe = Array.from({ length: 20 }, (_, i) => ({
  label: (i + 32).toString(),
  value: i + 18,
}));
const figures = [
  { label: "Худощавая", value: "slim" },
  { label: "Средняя", value: "medium" },
  { label: "Спортивная", value: "athletic" },
  { label: "Пышная", value: "curvy" },
];
const emit = defineEmits(["update", "next", "prev"]);
const selectBreast = [
  { label: "Первый размер", value: "1" },
  { label: "Второй размер", value: "2" },
  { label: "Третий размер", value: "3" },
  { label: "Четвертый размер", value: "4" },
  { label: "Пятый размер", value: "5" },
  { label: "Шестой размер", value: "6" },
];
const hairColors = [
  { label: "Блондинка", value: "blond", hex: "#fcebbd" },
  { label: "Брюнетка", value: "black", hex: "#2f2f2f" },
  { label: "Шатенка", value: "brown", hex: "#8b5a2b" },
  { label: "Рыжая", value: "red", hex: "#c1440e" },
  { label: "Крашеная", value: "dyed", hex: "#d474d4" },
];
const selectNationality = [
  { value: 1, label: "Белоруска" },
  { value: 2, label: "Казашка" },
  { value: 3, label: "Китаянка" },
  { value: 4, label: "Кореянка" },
  { value: 5, label: "Киргизка" },
  { value: 6, label: "Русская" },
  { value: 7, label: "Татарка" },
  { value: 8, label: "Украинка" },
  { value: 9, label: "Негритянка" },
  { value: 10, label: "Кабардинка" },
  { value: 11, label: "Узбечка" },
  { value: 11, label: "Армянка" },
];
const selectAppearance = [
  { value: 1, label: "Мулатка" },
  { value: 2, label: "Славянская" },
  { value: 3, label: "Татарка" },
  { value: 4, label: "Китаянка" },
  { value: 5, label: "Киргизка" },
  { value: 6, label: "Казашка" },
  { value: 7, label: "Метиска" },
  { value: 8, label: "Азиатская" },
  { value: 9, label: "Африканская" },
  { value: 10, label: "Кавказская" },
  { value: 11, label: "Латино" },
];
const selectClothingSize = [
  { label: "XS", value: "1" },
  { label: "S", value: "2" },
  { label: "M", value: "3" },
  { label: "L", value: "4" },
  { label: "XL", value: "5" },
  { label: "больше XL", value: "6" },
];
const selectIntimHaircut = [
  { value: 1, label: "Полная депиляция" },
  { value: 2, label: "Аккуратная стрижка" },
  { value: 3, label: "Натуральная" },
];

const localForm = reactive({
  not_younger: props.formData.not_younger || 18,
  not_older: props.formData.not_older || 55,
  age: props.formData.age || "",
  height: props.formData.height || "",
  weight: props.formData.weight || "",
  breast: props.formData.breast || "",
  clothing_size: props.formData.clothing_size || "",
  shoe_size: props.formData.shoe_size || "",
  nationality: props.formData.nationality || "",
  appearance: props.formData.appearance || "",
  intim_haircut: props.formData.intim_haircut || "",
  hair_color: props.formData.hair_color || "",
  body_type: props.formData.body_type || "",
  photo: props.formData.photo || [],
  video: props.formData.video || "",
});

watch(
  () => localForm,
  () => {
    emit("update", { ...localForm });
  },
  { deep: true }
);

const ageRange = reactive([
  props.formData.not_younger || 18,
  props.formData.not_older || 66,
]);


const localAgeRange = computed({
  get() {
    return [localForm.not_younger, localForm.not_older];
  },
  set(val) {
    localForm.not_younger = val[0];
    localForm.not_older = val[1];
  }
});


function emitNext() {
  emit("next");
}
</script>
<style scoped>
.title__c {
  color: #555;
}
.content__form {
  max-width: 900px;
  background: #fff;
  padding: 30px 40px;
  margin-bottom: 100px;
  padding-bottom: 100px;
}
.content {
  display: flex;
  gap: 15px;
  margin-bottom: 15px;
}
.content-col {
  display: flex;
  flex-direction: column;
  gap: 15px;
}
.params {
  display: grid;
  grid-template-columns: repeat(3, minmax(200px, 250px));
  gap: 16px;
}
@media (max-width: 900px) {
  .params {
    grid-template-columns: repeat(2, minmax(200px, 1fr));
    gap: 12px;
  }
}

@media (max-width: 600px) {
  .params {
    grid-template-columns: 1fr;
    gap: 10px;
  }

  .content__form {
    padding: 20px 16px 80px;
    max-width: 100%;
  }
}
</style>
