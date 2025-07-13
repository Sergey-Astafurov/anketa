<template>
  <div class="preview__section" v-if="tariffs?.length">
    <div class="tariff-group">
      <div v-if="dayTariffs.length">
        <div class="tariff-cards sexy">
          <div
            v-for="tariff in dayTariffs"
            :key="tariff.id"
            class="tariff-card sexy day"
          >
            <div class="tariff-top">
              <span class="tariff-time">{{ tariff.label }}</span>
            </div>
            <div class="tariff-price">
              <p>У тебя — <span>{{ formatPrice(tariff.you) }}</span></p>
              <p>У него — <span>{{ formatPrice(tariff.me) }}</span></p>
            </div>
          </div>
        </div>
      </div>
      <div v-if="nightTariffs.length">
        <div class="tariff-cards sexy">
          <div
            v-for="tariff in nightTariffs"
            :key="tariff.id"
            class="tariff-card sexy night"
          >
            <div class="tariff-top">
              <span class="tariff-time">{{ tariff.label }}</span>
            </div>
            <div class="tariff-price">
              <p>У тебя — <span>{{ formatPrice(tariff.you) }}</span></p>
              <p>У него — <span>{{ formatPrice(tariff.me) }}</span></p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {computed} from 'vue';
const props = defineProps({
  tariffs: Array,
});

const dayTariffs = computed(() => (props.tariffs || []).filter((t) => !t.isNight));
const nightTariffs = computed(() => (props.tariffs || []).filter((t) => t.isNight));

function formatPrice(val) {
  if (!val) return "-";
  const num = parseInt(val.toString().replace(/\D/g, ""));
  return isNaN(num) ? val : num.toLocaleString("ru-RU") + " ₽";
}
</script>

<style scoped>
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
.tariff-group {
  margin-top: 12px;
}
.tariff-cards.sexy {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-top: 14px;
}
.tariff-card.sexy {
  flex: 1 1 200px;
  background: linear-gradient(145deg, #3d2c41, #271c2e);
  border: 1px solid #5e3b6b;
  border-radius: 18px;
  padding: 18px;
  color: #f3d9ff;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.35);
  backdrop-filter: blur(6px);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
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
</style>
