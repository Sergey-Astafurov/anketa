<template>
  <div class="tabs" ref="tabRef">
    <div class="tabs__buttons">
      <button
        v-for="(cat, index) in categories"
        :key="cat.name"
        class="tab tabs__btn"
        :class="{ active: activeTab === index }"
        @click="setActive(index, $event)">
        {{ cat.name }}
      </button>
      <span class="menu__underline" :style="underlineStyle"></span>
      <div class="under"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, nextTick, onMounted, onBeforeUnmount } from 'vue';

const props = defineProps({
  categories: Array,
  modelValue: Number,
});
const emit = defineEmits(['update:modelValue']);

const activeTab = ref(props.modelValue ?? 0);
const tabRef = ref(null);
const underlineStyle = ref({ left: '0px', width: '0px' });

let isMounted = false; // флаг для контроля первого рендера

function setUnderline(el) {
  if (!el || !tabRef.value) return;
  const rect = el.getBoundingClientRect();
  const parentRect = tabRef.value.getBoundingClientRect();
  underlineStyle.value = {
    left: `${rect.left - parentRect.left}px`,
    width: `${rect.width}px`,
  };
}

function setActive(index, event) {
  if (activeTab.value === index) return;
  activeTab.value = index;
  emit('update:modelValue', index);
  setUnderline(event.target);
}

function handleResize() {
  nextTick(() => {
    const activeEl = tabRef.value?.querySelector('.tabs__btn.active');
    if (activeEl) setUnderline(activeEl);
  });
}

onMounted(() => {
  isMounted = true;
  nextTick(() => {
    const activeEl = tabRef.value?.querySelector('.tabs__btn.active');
    if (activeEl) setUnderline(activeEl);
  });
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
});

watch(
  () => props.modelValue,
  (newVal) => {
    if (newVal !== activeTab.value) {
      activeTab.value = newVal;
      nextTick(() => {
        const activeEl = tabRef.value?.querySelector('.tabs__btn.active');
        if (activeEl) setUnderline(activeEl);
      });
    }
  }
);

watch(
  activeTab,
  (newVal, oldVal) => {
    if (!isMounted) return; // Не выполнять до монтирования, чтобы избежать вызова лишнего рендера
    nextTick(() => {
      const activeEl = tabRef.value?.querySelector('.tabs__btn.active');
      if (activeEl) setUnderline(activeEl);
    });
  }
);
</script>



<style scoped>
.tabs {
  position: relative;
}
.tabs__buttons {
  position: relative;
  display: flex;
  gap: 20px;
  margin-bottom: 10px;
}
.tabs__buttons button {
  position: relative;
  padding: 8px 16px;
  border: none;
  cursor: pointer;
  font-size: 16px;
  border-radius: 4px;
  transition: all 0.3s ease;
}
.tabs__buttons button:hover {
  background: #e8e8e8;
}
.tabs__buttons button.active {
  /* active styles if needed */
}
.menu__underline {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 2px;
  background-color: #f90042 !important;
  width: 0;
  transition: left 0.3s, width 0.3s;
  z-index: 20;
}
.under {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 1px;
  width: 100%;
  background: #e8e8e8;
}

@media (max-width: 600px) {
  .tabs__buttons {
    flex-wrap: wrap;
    gap: 10px;
  }
  .tabs__buttons button {
    flex: 1 1 calc(50% - 10px);
    font-size: 14px;
    padding: 6px 10px;
  }
  .menu__underline {
    display: none; /* убрать подчеркивание на маленьких экранах, чтобы не ломать верстку */
  }
}

@media (max-width: 400px) {
  .tabs__buttons button {
    flex: 1 1 100%;
  }
}
</style>
