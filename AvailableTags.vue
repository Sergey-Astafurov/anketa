<template>
  <div class="tag-selector">
    <div v-for="group in availableTags" :key="group.category" class="tag-group">
      <div class="group-title">{{ group.category }}</div>
      <div class="tag-list">
        <button
          v-for="tag in group.tags"
          :key="tag"
          class="tag"
          :class="{ selected: modelValue.includes(tag) }"
          @click="toggleTag(tag)"
        >
          {{ tag }}
        </button>
      </div>
    </div>
    <div class="tag-count" :class="{ limit: modelValue.length === 7 }">
      Выбрано: {{ modelValue.length }} / 7
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  modelValue: {
    type: Array,
    default: () => [],
  },
});
const emit = defineEmits(["update:modelValue"]);

const availableTags = [
  {
    category: "Интимные услуги",
    tags: [
      "Анальный секс", "Анилингус клиенту", "Минет в машине", "Куннилингус",
      "Секс в машине", "МБР", "Страпон", "Рабыня", "Групповой секс",
      "Лесбийский секс", "Горловой минет", "МЖМ", "Садо мазо",
      "Доминирование", "Окончание на лицо", "Окончание на грудь",
      "Глубокий минет", "Фингеринг", "Порка", "Целуюсь"
    ],
  },
  {
    category: "Внешность и уход",
    tags: [
      "Есть тату", "Спортивная", "Молоденькая", "Милфа",
      "Силиконовая грудь", "Полная депиляция", "Аккуратная стрижка"
    ],
  },
  {
    category: "Условия приёма",
    tags: [
      "Апартаменты", "Выезд", "Есть подруга", "Эскорт", "Инди"
    ],
  },
];

function toggleTag(tag) {
  const newTags = [...props.modelValue];
  const index = newTags.indexOf(tag);
  if (index !== -1) {
    newTags.splice(index, 1);
  } else if (newTags.length < 7) {
    newTags.push(tag);
  }
  emit("update:modelValue", newTags);
}
</script>


<style scoped>
.tag-selector {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.tag-group {
  border-bottom: 1px solid #eee;
  padding-bottom: 16px;
}

.group-title {
  font-weight: 600;
  margin-bottom: 20px;
  font-size: 20px;
  color: #555;
}

.tag-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.tag {
  padding: 8px 17px;
  border-radius: 20px;
  border: 1.5px solid #555;
  background-color: transparent;
  color: #555;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.2s ease;
}

.tag:hover {
  border-color: #fc4d7b;
}

.tag.selected {
  color: #f90042;
  border-color: #f90042;
}

.tag-count {
  font-size: 14px;
  text-align: right;
  color: #555;
}

.tag-count.limit {
  color: #f90042;
  font-weight: 600;
}
@media (max-width: 600px) {
  .tag-selector {
    gap: 16px;
  }

  .group-title {
    font-size: 16px;
    margin-bottom: 12px;
  }

  .tag-list {
    gap: 8px;
  }

  .tag {
    font-size: 13px;
    padding: 8px 14px;
  }

  .tag-count {
    font-size: 13px;
  }
}


</style>
