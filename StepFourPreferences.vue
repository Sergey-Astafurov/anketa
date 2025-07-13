<template>
  <FormBase>
    <FormTitle title="Выберите теги подходящие вам" />
<AvailableTags v-model="selectedTags" />
    <FormTitle title="Предпочтения" />
    <TabsCategories :categories="categories" v-model="activeTab" />
    <div class="tabs__content">
      <SubCategoryItem
        v-for="sub in categories[activeTab].subcategories"
        :key="sub"
        :subcategory="sub"
        v-model:modelValue="selectedSubcategoriesMap[sub]"
        v-model:commentValue="comments[sub]"
        @update:modelValue="(val) => onSubcategoryToggle(sub, val)"
        @update:commentValue="(val) => onCommentChange(sub, val)" />
    </div>

    <div class="d-flex justify-content-between mt-4">
      <ButtonOutline @click="$emit('prev')" label="Назад" />
      <ButtonBase label="Далее" @click="emitNext" />
    </div>
  </FormBase>
</template>

<script setup>
import { ref, watch, reactive, computed, onMounted } from "vue";
import FormBase from "@/components/ui/Forms/FormBase.vue";
import AvailableTags from "@/components/ui/AvailableTags.vue";
import FormTitle from "@/components/ui/Forms/FormTitle.vue";
import ButtonBase from "@/components/ui/ButtonBase.vue";
import ButtonOutline from "@/components/ui/ButtonOutline.vue";
import TabsCategories from "@/components/ui/Anketa/TabsCategories.vue";
import SubCategoryItem from "@/components/ui/Anketa/SubCategoryItem.vue";

const props = defineProps({
  formData: Object,
});
const emit = defineEmits(["update", "next", "prev"]);
// Инициализация реактивных переменных
const selectedTags = ref(props.formData.tags || []);
const selectedSubcategories = ref(props.formData.selectedSubcategories || []);
const comments = reactive(props.formData.preferenceComments || {});
const activeTab = ref(0);
const categories = [
  {
     key: "sex",
    name: "Секс",
    subcategories: [
      "Секс классический",
      "Секс анальный",
      "Секс групповой",
      "Секс лесбийский",
      "Услуги семейной паре",
      "Минет в презервативе",
      "Минет глубокий",
      "Кунилингус",
      "Игрушки",
      "Окончание на грудь",
      "Окончание на лицо",
      "Окончание в рот",
      "Минет без презерватива",
      "Целуюсь",
      "Поза 69",
      "Сопровождение",
    ],
  },
  {
    key: "massage",
    name: "Массаж",
    subcategories: [
      "Расслабляющий массаж",
      "Эротический массаж",
      "Профессиональный массаж",
      "Урологический массаж",
      "Массаж семейной паре",
      "Массаж класический",
    ],
  },
  {
    key: "strip",
    name: "Стриптиз",
    subcategories: [
      "Стриптиз профи",
      "Стриптиз не профи",
      "Лесби откровенное",
      "Лесби - шоу легкое",
    ],
  },
  {
    key: "extreme",
    name: "Экстрим",
    subcategories: [
      "Страпон",
      "Анилингус вам",
      "Анилингус мне",
      "Золотой дождь вам",
      "Золотой дождь мне",
      "Фистинг анальный вам",
      "Фистинг анальный мне",
      "Фистинг классический",
      "Фингеринг",
    ],
  },
  {
    key: "sado",
    name: "Садо-мазо",
    subcategories: [
      "Госпожа",
      "Рабыня",
      "Доминация",
      "Бандаж",
      "Порка",
      "Фетиш",
      "Трамплинг",
      "Игры",
      "Шибари",
      "Фейсситтинг",
      "Подчинение",
    ],
  },
  {
    key: "other",
    name: "Разное",
    subcategories: [
      "Клизма",
      "GFE",
      "Есть загранпаспорт",
      "Фото или видео съёмка",
      "Виртуальный секс",
      "Секс по телефону",
      "Ролевые игры",
      "Готова к поездкам в другой город",
      "Есть шенген",
    ],
  },
];
// Для удобного биндинга: объект, где ключ — подкатегория, значение — true/false
const selectedSubcategoriesMap = computed(() => {
  const map = {};
  for (const cat of categories) {
    for (const sub of cat.subcategories) {
      map[sub] = selectedSubcategories.value.includes(sub);
    }
  }
  return map;
});

function onSubcategoryToggle(sub, isSelected) {
  const idx = selectedSubcategories.value.indexOf(sub);
  if (isSelected && idx === -1) {
    selectedSubcategories.value.push(sub);
  } else if (!isSelected && idx !== -1) {
    selectedSubcategories.value.splice(idx, 1);
    if (comments[sub]) delete comments[sub];
  }
}

function onCommentChange(sub, val) {
  comments[sub] = val;
}
// Следим за изменениями и сохраняем в localStorage
function emitNext() {
  const service = {};
  categories.forEach(cat => {
    service[cat.key] = [];
  });

  for (const cat of categories) {
    for (const sub of cat.subcategories) {
      if (selectedSubcategories.value.includes(sub)) {
        service[cat.key].push({
          name: sub,
          description: comments[sub] || "",
        });
      }
    }
  }

  emit("update", {
    tags: selectedTags.value,
    service,
  });
  emit("next");
}
</script>


<style scoped>
.tabs__content {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 10px;
}
</style>
