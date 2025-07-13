<template>
  <FormBase>
    <FormTitle title="Основная информация" />
    <div class="wrapp">
      <div class="params">
        <TheInput
          name="name"
          label="Имя"
          id="firstName"
          :error="errors.firstName"
          v-model="localForm.name" />
        <TheInput
          name="telephone"
          label="Номер телефона"
          id="telephone"
          :maska="{ mask: '+{7}(000)000-00-00' }"
          :error="errors.telephone"
          v-model="localForm.tel" />

        <TheInput
          name="telega"
          label="Telegram"
          id="telega"
          :error="errors.telega"
          :maska="{
            mask: '{@}***************************',
          }"
          v-model="localForm.tg" />
        <TheInput
          name="wts"
          label="WhatsApp"
          id="whatsApp"
          :error="errors.whatsApp"
          :maska="{ mask: '+{7}(000)000-00-00' }"
          v-model="localForm.whatsapp" />
        <TheHintInput
          name="city"
          :array="cities"
          :error="errors.city"
          id="city"
          @select="addLocationHandler"
          v-model="localForm.city"
          @citynames="getAllCity"
          :current="localForm.city"
          label="Ваш город" />
        <TheHintInput
          :array="districts"
          id="district"
          name="district"
          v-model="localForm.district"
          :error="errors.district"
          label="Район"
          @select="addLocationHandler"
          :current="localForm.district" />
        <TheHintInput
          name="metro"
          id="metro"
          :array="metros"
          :error="errors.metro"
          v-model="localForm.metro"
          @select="addLocationHandler"
          :current="localForm.metro"
          label="Метро" />
      </div>
      <div class="textarea_text mb-3">
        <TheTextArea
          name="about"
          id="about"
          :error="errors.about"
          label="О себе"
          v-model="localForm.description" />
      </div>
    </div>

    <div class="d-flex btn__wrapp justify-content-end">
      <ButtonBase label="Далее" @click="emitNext" />
    </div>
  </FormBase>
</template>

<script setup>
import axios from "axios";
import { reactive, watch, ref, onMounted } from "vue";
import TheInput from "../ui/Forms/TheInput.vue";
import TheHintInput from "../ui/Forms/TheHintInput.vue";
import TheTextArea from "@/components/ui/Forms/TheTextArea.vue";
import FormBase from "@/components/ui/Forms/FormBase.vue";
import FormTitle from "@/components/ui/Forms/FormTitle.vue";
import ButtonBase from "@/components/ui/ButtonBase.vue";

import { useStore } from "vuex";
import { computed } from "vue";

const store = useStore();
const user = computed(() => store.getters.getUser);
const apiUrl = computed(() => store.getters.getApiUrl);
const apiDomine = computed(() => store.getters["getApiDomine"]);

const props = defineProps({
  formData: Object,
});
const emit = defineEmits(["update", "next"]);



const cities = ref([]);
const metrosByCity = ref([]);
const metros = ref([]);
const metroName = ref("");
const districts = ref([]);

const localForm = reactive({
  name: props.formData.name || "",
  tel: props.formData.tel || "",
  tg: props.formData.tg || "",
  whatsapp: props.formData.whatsapp || "",
  description: props.formData.description || "",
  city: props.formData.city || "",
  city_id: props.formData.city_id || "",
  district: props.formData.district || "",
  district_id: props.formData.district_id || "",
  metro: props.formData.metro || "",
  metro_id: props.formData.metro_id || "",
});

async function addLocationHandler(data) {
  if (!data) return;

  // Если выбрали город
  if ("country_id" in data) {
    localForm.city_id = data.id;
    localForm.city = data.name;
    metros.value = data.metro || [];
    districts.value = data.districtWithMetro || [];


    return;
  }

  // Если выбрано метро + район (обычно из метро)
  if ("city_id" in data && "district_id" in data) {
    localForm.metro_id = data.id;
    localForm.metro = data.name;
    localForm.district_id = data.district_id;

    // Найдём район по ID, если возможно
    const districtObj = districts.value.find((d) => d.id === data.district_id);
    if (districtObj) {
      localForm.district = districtObj.name;
    }
    return;
  }

  // Если выбран только район
  if ("city_id" in data && !("district_id" in data)) {
    localForm.district_id = data.id;
    localForm.district = data.name;
    return;
  }
}

function getAllCity(param) {
  if (param.length >= 1) {
    axios
      .get(
        `${apiUrl.value}api-hint/city&auth=${user.value.username}:${user.value.auth_key}&str=${param}`
      )
      .then((response) => {
        if (response.data.status !== false) {
          cities.value = response.data;

          if (localForm.city_id) {
            const currentCity = cities.value.find(
              (c) => c.id === localForm.city_id
            );
            if (currentCity) {
              addLocationHandler(currentCity);

              if (localForm.metro_id) {
                metroName.value =
                  currentCity.metro?.find((m) => m.id === localForm.metro_id)
                    ?.name || "";
              }

              if (localForm.district_id) {
                const currentDistrict = currentCity.districtWithMetro?.find(
                  (d) => d.id === localForm.district_id
                );
                if (currentDistrict) {
                  localForm.district = currentDistrict.name;
                }
              }
            }
          }
        }
      });
  }
}
// --- Получение метро по городу ---
function getMetroByCityId(id) {
  axios
    .get(
      `${apiUrl.value}api/get-metro-by-city-id&auth=${user.value.username}:${user.value.auth_key}&id=${id}`
    )
    .then((response) => {
      metrosByCity.value = response.data;
      const match = metrosByCity.value.find(
        (m) => m.id === formData.value.metro_id
      );
      if (match) {
        metroName.value = match.name;
      }
    });
}
function hintMetro() {
  if (metroName.value.length > 1) {
    const param =
      user.value.username +
      ":" +
      user.value.auth_key +
      "&city_id=" +
      formData.value.city_id +
      "&str=" +
      metroName.value;

    axios
      .get(apiUrl.value + "api-hint/metro&auth=" + param)
      .then((response) => {
        if (!response.data.status) {
          errorHintMetro.value = response.data.error;
        }
        metrosByCity.value = response.data;
        hMetro.value = true;
      });
  }
}
// --- Выбор метро из подсказки ---
function selectHintMetro(m) {
  formData.value.metro_id = m.id;
  metroName.value = m.name;
  hMetro.value = false;
}

const errors = reactive({
  name: "",
  tel: "",
  tg: "",
  whatsApp: "",
  description: "",
  city: "",
  district: "",
  metro: "",
});

function validateForm() {
  let isValid = true;
  // Имя — обязательно, только русские буквы
  if (!formData.name || !/^[А-Яа-яЁё]+$/.test(formData.name)) {
    errors.firstName = "Введите имя на русском";
    isValid = false;
  } else {
    errors.firstName = "";
  }
  // Телефон — обязательно
  if (!formData.tel || formData.tel.length < 17) {
    errors.telephone = "Введите корректный номер";
    isValid = false;
    // console.log(formData.telephone.length);
  } else {
    errors.telephone = "";
  }
  // Telegram — по желанию, но если есть, то валидный
  if (formData.tg && !/^@[\w\d_]{3,32}$/.test(formData.tg)) {
    errors.telega = "Неверный Telegram";
    isValid = false;
  } else {
    errors.telega = "";
  }
  // whatsApp — как телефон
  if (!formData.whatsapp || formData.whatsapp.length < 17) {
    errors.whatsApp = "Введите номер whatsApp";
    isValid = false;
  } else {
    errors.whatsApp = "";
  }
  // О себе — не обязательно, но ограничим
  if (formData.description && formData.description.length > 500) {
    errors.about = "Макс. 500 символов";
    isValid = false;
  } else {
    errors.about = "";
  }
  // Город/район/метро
  if (!formData.city_id) {
    errors.city = "Выберите город";
    isValid = false;
  } else {
    errors.city = "";
  }
  return isValid;
}

function emitNext() {
  emit("update", localForm);
  emit("next");
  window.scrollTo({ top: 0, behavior: "smooth" });
}
onMounted(() => {
  if (localForm.city_id) {
    // передаём хоть какой-то символ, чтобы `getAllCity` отработал
    getAllCity(localForm.city);
  }
  console.log(localForm);
});
</script>

<style scoped>
.params {
  display: grid;
  grid-template-columns: repeat(2, minmax(200px, 250px));
  gap: 16px;
  margin-bottom: 16px;
}
.textarea_text {
  max-width: 516px;
  width: 100%;
}
.btn__wrapp {
  max-width: 516px;
  width: 100%;
  margin: 0 auto;
}
.wrapp {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
.content__form {
  display: flex;
  flex-direction: column;
  gap: 30px;
  margin-bottom: 30px;
}
@media (max-width: 768px) {
  .params {
    grid-template-columns: 1fr !important;
    gap: 12px !important;
  }

  .textarea_text {
    max-width: 100% !important;
  }

  .btn__wrapp {
    max-width: 100% !important;
    padding: 0 16px;
  }

  .wrapp {
    padding: 0 16px;
  }
}

@media (max-width: 480px) {
  .form_one {
    padding: 16px 16px;
  }
  .params {
    gap: 8px !important;
  }
  .wrapp {
    align-items: stretch !important;
  }
  .btn__wrapp {
    justify-content: center !important;
  }
}
</style>
