<template>
  <div class="max-w-xl mx-auto p-4">
    <keep-alive>
      <component
        :is="currentStepComponent"
        :form-data="formData"
        @update="updateForm"
        @next="nextStep"
        @submit="addProfile"
        @prev="prevStep" />
    </keep-alive>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch, onMounted } from "vue";
import StepOneBasicInfo from "StepOneBasicInfo.vue";
import StepTwoCharacteristics from "StepTwoCharacteristics.vue";
import StepFourPreferences from "StepFourPreferences.vue";
import StepFivePreview from "StepFivePreview.vue";
import StepFourTarifs from "StepFourTarifs.vue";
import { useStore } from "vuex";
import axios from "axios";
import StepThreePhotoVideo from "StepThreePhotoVideo.vue";
import { useRoute } from "vue-router";

const route = useRoute();
const store = useStore();

const user = computed(() => store.getters.getUser);
const apiUrl = computed(() => store.getters.getApiUrl);
const apiDomine = computed(() => store.getters["getApiDomine"]);

onMounted(() => {
  if (route.params && route.params.id) {
    getGirlAnketa(route.params.id);
  } else if (anketaId !== 1) {
    getGirlAnketa(anketaId);
  } else if (user.value?.profile?.user_id && user.value?.user_id) {
    formData.user_id = +user.value.profile.user_id;
    formData.author_id = +user.value.user_id;
    formData.status_id = 1;
  }
});
const anketaId = 1;

const step = ref(1);
const saved = localStorage.getItem("anketa-form-data");
const formData = reactive(
  saved
    ? JSON.parse(saved)
    : {
        category_id: 8,
        user_id: "",
        name: "",
        metro: "",
        metro_id: "",
        district_id: "",
        description: "",
        tel: "",
        tg: "",
        city_id: "",
        city: "",
        whatsapp: "",
        age: "",
        height: "",
        weight: "",
        breast: "",
        clothing_size: "",
        shoe_size: "",
        not_younger: "",
        not_older: "",
        hair_color: "",
        intim_haircut: "",
        appearance: "",
        nationality: "",
        body_type: "",
        price_hour_app: "",
        price_two_hour_app: "",
        price_night_app: "",
        price_hour_night_app: "",
        price_night_exit: "",
        price_hour_night_exit: "",
        price_hour_exit: "",
        price_two_hour_exit: "",
        currency_id: "1",
        service: {
          sex: [],
          dop: [],
          bdsm: [],
          massage: [],
          strip: [],
          extreme: [],
        },
        checkService: [],
        status_id: 7,
        photo: [],
        video: "",
        tags: [],
        verification_code: "",
        verification_video: "",
      }
);

if (user.value && user.value.id) {
  formData.user_id = user.value.id;
}

watch(
  user,
  (newUser) => {
    if (newUser && newUser.id) {
      formData.user_id = newUser.id;
    }
  },
  { immediate: true }
);

const steps = [
  StepOneBasicInfo,
  StepTwoCharacteristics,
  StepThreePhotoVideo,
  StepFourTarifs,
  StepFourPreferences,
  StepFivePreview,
];

const currentStepComponent = computed(() => steps[step.value - 1]);

function nextStep() {
  if (step.value < steps.length) step.value++;
}
function prevStep() {
  if (step.value > 1) step.value--;
}

function updateForm(updated) {
  Object.assign(formData, updated);
}

watch(
  formData,
  (newData) => {
    localStorage.setItem("anketa-form-data", JSON.stringify(newData));
  },
  { deep: true }
);

const auth = `${user.value.username}:${user.value.auth_key}`;
// Примерные значения, можешь заменить своими

async function addProfile() {
  // Собираем checkService

  // // Проверка возраста (если используешь возрастное ограничение)
  // if (formData.age && Number(formData.age) < 18) {
  //   const message = {
  //     title: "Ошибка отправки анкеты",
  //     massage: "Возраст не может быть меньше 18 лет",
  //     type: "danger"
  //   };
  //   store.commit("setFlashMessage", message);
  //   return;
  // }

  // // Проверка обязательных полей
  // const requiredFields = ["name", "tel", "whatsapp", "tg", "city_id"];
  // const hasEmpty = requiredFields.some(key => !formData[key] || formData[key] === "");

  // if (hasEmpty) {
  //   const message = {
  //     title: "Ошибка отправки анкеты",
  //     massage: "Заполнены не все обязательные поля",
  //     type: "danger"
  //   };
  //   store.commit("setFlashMessage", message);
  //   return;
  // }

  // Проверка тарифов — минимум 3000
  // const invalidTariff = Object.values(formData.tariffs).some(
  //   t => (t.you && t.you < 3000) || (t.me && t.me < 3000)
  // );

  // if (invalidTariff) {
  //   const message = {
  //     title: "Ошибка отправки анкеты",
  //     massage: "Минимальная сумма тарифа — 3000 рублей",
  //     type: "danger"
  //   };
  //   store.commit("setFlashMessage", message);
  //   return;
  // }

  // Приведение телефона к числу без символов
  // const clearPhone = tel => tel.replace(/\D/g, "");
  // formData.tel = clearPhone(formData.tel);
  // formData.whatsapp = clearPhone(formData.whatsapp);

  // Формируем payload
  // const payload = {
  //   user_id: 1,
  //   category_id: formData.category_id || 1,
  //   name: formData.name,
  //   age: formData.age,
  //   height: formData.height,
  //   weight: formData.weight,
  //   breast: formData.breast,
  //   hair_color: formData.hair_color,
  //   metro: formData.metro,
  //   description: formData.description,
  //   tel: Number(formData.tel.replace(/\D/g, "")),
  //   tg: formData.tg,
  //   city_id: formData.city_id,
  //   district_id: formData.district_id,
  //   metro_id: formData.metro_id,
  //   whatsapp: Number(formData.whatsapp.replace(/\D/g, "")),
  //   clothing_size: formData.clothing_size,
  //   shoe_size: formData.shoe_size,
  //   intim_haircut: String(formData.intim_haircut),
  //   appearance: String(formData.appearance),
  //   nationality: String(formData.nationality),
  //   age_range: formData.age_range,
  //   preferences: formData.preferences,
  //   tariffs: formData.tariffs,
  //   checkService: formData.checkService || [],
  //   currency_id: formData.currency_id || 1,
  //   status_id: 7,
  //   category_id: formData.category_id || 8,
  //   currency_id: formData.currency_id || "1",
  //   status_id: formData.status_id || 7,
  //   preferences: formData.preferences,
  //   selectedSubcategories: formData.selectedSubcategories,
  //   preferenceComments: formData.preferenceComments,
  // };
  formData.photo = (formData.photo || [])
    .filter((p) => typeof p === "object" && p.name)
    .map((p) => ({
      name: p.name.trim().replace(/\/{2,}/g, "/"),
      preview: p.preview,
    }));
  console.log(formData.photo);
  // Главное фото — из исправленных путей
  const mainPic =
    typeof formData.photo[0] === "string"
      ? formData.photo[0]
      : formData.photo[0]?.name || "";
  console.log(formData.photo[0].name);
  const payload = {
    user_id: formData.user_id || "",
    pic: mainPic,
    category_id: formData.category_id || 8,
    name: formData.name || "",
    age: formData.age || "",
    height: formData.height || "",
    weight: formData.weight || "",
    breast: formData.breast || "",
    clothing_size: formData.clothing_size || "",
    shoe_size: formData.shoe_size || "",
    not_younger: formData.not_younger || "",
    not_older: formData.not_older || "",
    hair_color: formData.hair_color || "",
    intim_haircut: String(formData.intim_haircut || ""),
    appearance: String(formData.appearance || ""),
    nationality: String(formData.nationality || ""),
    body_type: formData.body_type || "",

    metro: formData.metro || "",
    metro_id: formData.metro_id || "",
    district_id: formData.district_id || "",
    city_id: formData.city_id || "",
    city: formData.city || "",

    price_hour_app: formData.price_hour_app || "",
    price_two_hour_app: formData.price_two_hour_app || "",
    price_night_app: formData.price_night_app || "",
    price_hour_night_app: formData.price_hour_night_app || "",
    price_night_exit: formData.price_night_exit || "",
    price_hour_night_exit: formData.price_hour_night_exit || "",
    price_hour_exit: formData.price_hour_exit || "",
    price_two_hour_exit: formData.price_two_hour_exit || "",

    description: formData.description || "",

    tel: Number(formData.tel?.replace(/\D/g, "")) || "",
    tg: formData.tg || "",
    whatsapp: Number(formData.whatsapp?.replace(/\D/g, "")) || "",

    currency_id: formData.currency_id || "1",
    checkService: formData.checkService || [],
    status_id: formData.status_id || 7,

    photo: formData.photo.map((p) => p.name || ""),
    video:
      typeof formData.video === "object" && formData.video.name
        ? formData.video.name
        : formData.video,
    tags: formData.tags || [],

    verification_code: formData.verification_code || "",
    verification_video: formData.verification_video || "",
  };
  console.log(payload.pic);

  // Отправка
  try {
    const auth = `${user.value.username}:${user.value.auth_key}`;
    const url = `${apiUrl.value}api-girl/set-draft-profile&auth=${auth}`;
    const response = await axios.post(url, payload);
    console.log("Успешно отправлено:", response.data);
    // localStorage.removeItem("anketa-form-data");

    // store.commit("setFlashMessage", {
    //   title: "Успешно",
    //   massage: "Анкета сохранена как черновик",
    //   type: "success"
    // });
  } catch (error) {
    console.error("Ошибка отправки анкеты", error?.response?.data || error);

    store.commit("setFlashMessage", {
      title: "Ошибка",
      massage: error?.response?.data?.error
        ? JSON.stringify(error.response.data.error)
        : "Произошла ошибка при сохранении",
      type: "danger",
    });
  }
}
</script>
