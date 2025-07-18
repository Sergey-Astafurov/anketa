<script>
import { ref } from "vue";
// Import Swiper Vue.js components
import { Swiper, SwiperSlide } from "swiper/vue";
import {
  FreeMode,
  Navigation,
  Thumbs,
  Pagination,
  EffectFade,
  Mousewheel,
} from "swiper/modules";
//import Chat from '@/components/blocks/Chat.vue'

import "swiper/css/free-mode";
import "swiper/css/navigation";
import "swiper/css/thumbs";
import "swiper/swiper-bundle.css";
import "@/styles/swiper.css";
import "swiper/css/effect-fade";
import axios from "axios";

import Chat from "@/components/blocks/Chat.vue";
//import Chat2 from '@/components/blocks/Chat-old.vue'
import VideoIndividually from "@/components/ui/UiVideoIndividually.vue";
//import BreadCrumbs from '@/components/ui/BreadCrumbs.vue'
import Fancybox from "@/components/ui/FancyBoxGallery.vue";
import Feedback from "@/components/blocks/Feedback.vue";
import updateStat from "@/mixins/updateStat";
import conjugation from "@/mixins/conjugation";
import { useCookies } from "vue3-cookies";
import LikeIcon from "@/components/icons/LikeIcon.vue";
import LogInCard from "@/components/blocks/LogInCard.vue";
import AllTags from "@/components/ui/common/tags/AllTags.vue";
import MyLife from "@/components/ui/mylife/MyLife.vue";
import telegramInputs from "@/mixins/posttg";
import router from "@/router/router";
import FormBase from "@/components/ui/Forms/FormBase.vue";
import FormTitle from "@/components/ui/Forms/FormTitle.vue";
export default {
  components: {
    FormTitle,
    FormBase,
    LogInCard,
    LikeIcon,
    Feedback,
    Swiper,
    SwiperSlide,
    VideoIndividually,
    Chat,
    //BreadCrumbs,
    Fancybox,
    AllTags,
    MyLife,
    //Feedback
  },
  mixins: [updateStat, conjugation, telegramInputs],
  data() {
    return {
      isLiked: false,
      user: this.$store.getters.getUser,
      apiUrl: this.$store.getters.getApiUrl,
      apiDomine: this.$store.getters.getApiDomine,
      /*breadCrumbs: [
                {
                    type: 'link',
                    link: '/',
                    name: 'Главная'
                }
            ],*/
      optionsFancyBox: {
        Carousel: {
          infinite: true,
        },
      },
      like: "#fff",
      imgList: [],
      path: [],
      service: {
        sex: [],
        massage: [],
        strip: [],
        extreme: [],
        bdsm: [],
        dop: [],
      },
      city: {},
      copyAlert: "",
      show: false,
      currency: 300,
      girlAnketa: {
        tg: "",
        photo: [],
      },
      swiperOptions: {
        breakpoints: {
          breakpoints: {
            374: { slidesPerView: 2 },
            1240: { slidesPerView: 4 },
          },
        },
      },
      activeServiceTab: {
        service: "",
        className: "active",
        isUsed: false,
      },
      loginTitle: "",
      openModal: false,
      positionModal: false,
      methodChangeStatusUrl: "api-girl/update-status",
      activOrOff: null,
    };
  },
  setup() {
    const thumbsSwiper = ref(null);

    const setThumbsSwiper = (swiper) => {
      thumbsSwiper.value = swiper;
    };

    const { cookies } = useCookies();

    return {
      cookies,
      thumbsSwiper,
      setThumbsSwiper,
      modules: [
        FreeMode,
        Navigation,
        Thumbs,
        Pagination,
        EffectFade,
        Mousewheel,
      ],
    };
  },
  methods: {
    addLike() {
      this.isLiked = !this.isLiked;
    },
    /**
     * Добавляет в куки ID текущей анкеты для последующего отображения в подборке Вы смотрели
     * @returns {boolean}
     */
    userViewProfile() {
      if (this.cookies.isKey("views")) {
        let views;
        views = JSON.parse(this.cookies.get("views"));

        for (let v of views) {
          if (parseInt(v) === parseInt(this.$route.params.id)) {
            return true;
          }
        }

        views.push(this.$route.params.id);

        this.cookies.set("views", JSON.stringify(views));
      } else {
        let views = [this.$route.params.id];
        this.cookies.set("views", JSON.stringify(views));
      }
    },
    toggleShow(event) {
      const element = event.target;
      if (element.classList.contains("is-hide")) {
        element.classList.remove("is-hide");
      } else {
        element.classList.add("is-hide");
      }
    },
    setTel(event) {
      const val = event.target.value.replace(/[^0-9]/g, "");
      event.target.value = val;
    },
    currencyBtn(currencyName) {
      this.currency = currencyName;
    },
    copyUrl() {
      const element = this.$refs.copyAlert;

      if ("clipboard" in navigator) {
        navigator.clipboard.writeText(this.anketaURL).then(
          function () {
            element.textContent = "Адрес страницы скопирован";
            element.classList.add("success");
          },
          function (err) {
            element.textContent = "Ошибка копирования: " + err;
            element.classList.add("error");
          }
        );
      } else {
        const textArea = document.createElement("textarea");

        textArea.value = this.anketaURL;
        document.body.appendChild(textArea);
        textArea.select();

        try {
          document.execCommand("copy");
          element.textContent = "Адрес страницы скопирован";
          element.classList.add("success");
        } catch (err) {
          element.textContent = "Ошибка копирования: " + err;
          element.classList.add("error");
        } finally {
          document.body.removeChild(textArea);
        }
      }
    },
    renderImgList(imgs) {
      imgs.forEach((item) => {
        this.imgList.push({
          url: this.apiDomine + `/web/uploads/girls/photo/${item.pic}`,
        });
      });
    },
    async getGirlAnketa() {
      axios
        .get(
          this.apiUrl +
            "api-girl/get-profile-by-id&auth=" +
            this.user.username +
            ":" +
            this.user.auth_key +
            "&id=" +
            this.$route.params.id +
            "&photo_size[width]=700" +
            "&photo_size[height]=700"
        )
        .then((response) => {
          let res = response.data;
          console.log("Фото из API:", response.data.photo);
          // console.log('URL фото:', this.apiDomine + `/web/uploads/girls/photo/${item.pic}`)
          if (res.activity.status == 1) {
            this.activOrOff = true;
          } else {
            this.activOrOff = false;
          }
          console.log("ответ получен, response.data:", response.data);
          this.girlAnketa = response.data;
          console.log("фото переданы в renderImgList:", response.data.photo);
          this.renderImgList(response.data.photo);
          this.getService();
          if (this.girlAnketa.isSaved) {
            this.like = "#FF4032";
          }
          this.m_updateView(this.girlAnketa.id, "view");

          if (this.girlAnketa.name && this.girlAnketa.city.name) {
            document.title = `Индивидуалка ${this.girlAnketa.name} ${this.girlAnketa.city.name} эскорт сервиса Egoza`;
          }
        })
        .catch((error) => console.log("getGirlAnketa: ", error));
    },
    getService() {
      for (let s of this.girlAnketa.checkService) {
        if (s.service?.category_id === 1) {
          this.service.sex.push(s);
        }
        if (s.service?.category_id === 2) {
          this.service.dop.push(s);
        }
        if (s.service?.category_id === 3) {
          this.service.bdsm.push(s);
        }
        if (s.service?.category_id === 4) {
          this.service.massage.push(s);
        }
        if (s.service?.category_id === 5) {
          this.service.strip.push(s);
        }
        if (s.service?.category_id === 6) {
          this.service.extreme.push(s);
        }
      }
    },

    addLike() {
      if (!this.user.isLogged) {
        this.loginTitle =
          "Что бы добавить анкету в избранное, войдите или зарегистрируйтесь в сервисе";
        this.show = true;
      }
      // if(this.user.name)
      if (this.girlAnketa.isSaved) {
        this.delSaved();
      } else {
        this.addSaved();
      }
    },
    addSaved() {
      axios
        .post(
          this.apiUrl +
            "api-user/save-girl&auth=" +
            this.user.username +
            ":" +
            this.user.auth_key,
          { user_id: this.user.user_id, profile_id: this.girlAnketa.id }
        )
        .then((response) => {
          if (response.data.status) {
            this.girlAnketa.isSaved = true;
            this.girlAnketa.stat.like++;
            this.like = "#FF4032";
          }
          this.getusersTg();
        })
        .catch((error) => console.log(error));
    },
    async getusersTg() {
      try {
        const res = await axios.get(
          `${this.apiUrl}api-user/get-user-settings&auth=${this.user.username}:${this.user.auth_key}&user_id=${this.girlAnketa.user_id}`
        );
        const data = res.data.data;
        data.find((el) => {
          if (el.name == "favourites" && el.value == "true") {
            if (this.girlAnketa.user_id != this.user.user_id) {
              this.telegramInputs(
                this.girlAnketa.user_id,
                "Вашу анкету добавили в избранное"
              );
            }
          }
        });
      } catch (error) {
        console.log("inputs.vue tg", error);
      }
    },
    delSaved() {
      axios
        .post(
          this.apiUrl +
            "api-user/unsave-girl&auth=" +
            this.user.username +
            ":" +
            this.user.auth_key,
          { user_id: this.user.user_id, profile_id: this.girlAnketa.id }
        )
        .then((response) => {
          if (response.data.status) {
            this.girlAnketa.isSaved = false;
            this.girlAnketa.stat.like--;
            this.like = "#fff";
          }
        })
        .catch((error) => console.log(error));
    },
    hideDialog() {
      this.show = false;
      this.$emit("successLogin");
    },
    async sendMessageClick() {
      if (this.user.isLogged) {
        const auth = "&auth=" + this.user.username + ":" + this.user.auth_key;
        const params =
          auth +
          "&user_id=" +
          this.user.user_id +
          "&target_user_id=" +
          this.girlAnketa.user_id;
        const { data } = await axios.get(
          this.apiUrl + "api-chat/get-private-chat" + params
        );
        let chatId = data.id || null;

        if (!data.id && !data.status) {
          const setChat = {
            type_id: 3,
            user_id: +this.user.user_id,
            target_user_id: +this.girlAnketa.user_id,
          };

          const { data } = await axios.post(
            this.apiUrl + "api-chat/set-chat" + auth,
            setChat
          );

          chatId = data.chat && data.chat.id ? data.chat.id : null;
        }

        const activeChatItem = {
          id: chatId,
          name: this.girlAnketa.name,
          recipient_id: +this.girlAnketa.user_id,
          message_id: new Date().getTime(),
        };

        console.log("activeChatItem:", activeChatItem);
        this.$store.commit("setActiveChat", activeChatItem);
      } else {
        this.loginTitle =
          "Что бы написать, войдите или зарегистрируйтесь в сервисе";
        this.show = true;
      }
    },

    confirmVideo() {
      const idParams = this.$route.params.id;

      let params = {
        status_id: 9,
        id: idParams,
      };

      axios
        .post(
          this.apiUrl +
            this.methodChangeStatusUrl +
            "&auth=" +
            this.user.username +
            ":" +
            this.user.auth_key,
          params
        )
        .then(() => {
          this.openModal = false;
        })
        .catch((error) => {
          console.log("произошла ошибка, при смене статуса" + error);
        });
    },

    rejectVideo() {
      const idParams = this.$route.params.id;

      let params = {
        status_id: 10,
        id: idParams,
      };

      axios
        .post(
          this.apiUrl +
            this.methodChangeStatusUrl +
            "&auth=" +
            this.user.username +
            ":" +
            this.user.auth_key,
          params
        )
        .then(() => {
          this.openModal = false;
        })
        .catch((error) => {
          console.log("произошла ошибка, при смене статуса" + error);
        });
    },
  },
  async mounted() {
    await this.getGirlAnketa();
    // console.log('Фото из API:', response.data.photo)

    axios.get(
      this.apiUrl +
        "api-girl/get-profiles-list-short-data&auth=" +
        this.user.username +
        ":" +
        this.user.auth_key
    );
    this.userViewProfile();

    console.log(this.girlAnketa);
  },
  computed: {
    slideLimit() {
      return this.girlAnketa.photo.slice(0, 5);
    },
    anketaURL() {
      return window.location.href;
    },
    lastActivity() {
      const timestamp = this.girlAnketa?.activity?.last_activity;
      if (timestamp) {
        const currentDate = new Date();
        const diffInMilliseconds = currentDate - timestamp;
        const diffInDays = diffInMilliseconds / (1000 * 60 * 60 * 24);

        if (diffInDays >= 3) {
          return "Недавно";
        }

        const date = new Date(timestamp);
        return date.toLocaleString("ru-RU").replace(",", "");
      } else {
        return "Недавно";
      }
    },
    girlDistrict() {
      const id = this.girlAnketa?.district_id;
      const cities = this.$store.getters.getCities;
      const city = cities.find((c) => c.id === this.girlAnketa.city_id);
      let district;
      if (city && city.id) {
        const disctrict = city.districtWithMetro.find((c) => c.id === id);
        return disctrict ? disctrict.name : "";
      }
      return "";
    },
    formatPhoneNumber() {
      const str = this.girlAnketa.tel.toString();
      const formattedNumber = str.replace(
        /(\d{1})(\d{3})(\d{3})(\d{4})/,
        "+$1 ($2) $3-$4"
      );
      return formattedNumber;
    },
    todayWork() {
      const today = new Date().toISOString().split("T")[0];
      const status = this.girlAnketa.schedule?.find((s) => s.date === today);

      return status && status.status ? status.status : false;
    },
    activeTab() {
      if (this.service.sex.length) {
        return "sex";
      }
      if (this.service.massage.length) {
        return "massage";
      }
      if (this.service.strip.length) {
        return "strip";
      }
      if (this.service.extreme.length) {
        return "extreme";
      }
      if (this.service.bdsm.length) {
        return "bdsm";
      }
      if (this.service.dop.length) {
        return "dop";
      }
    },
    // isOnline() {
    //     const currentTime = new Date().getTime()
    //     const timestamp = this.girlAnketa?.activity?.last_activity
    //     const difference = currentTime - timestamp
    //     return difference < 300000;
    // }
  },
};
</script>
<template>
  <div class="bg-main">
    <div class="main-container">
      <FormBase>
        <div class="anketa">
          <div
            v-if="false"
            class="anketa-top align-items-center justify-content-between py-3 row">
            <!-- <BreadCrumbs :breadCrumbs="breadCrumbs" class="col-7 col-lg-5"/> -->
            <div class="col-5 ms-auto text-end">
              <div class="anketa-top__url d-none d-lg-flex">
                <div class="me-auto ps-2">{{ anketaURL }}</div>
                <button type="button" @click="copyUrl">Копировать</button>
                <div class="copy-alert" ref="copyAlert"></div>
              </div>
              <div
                class="anketa-top__url anketa-top__url--mobile d-block d-lg-none">
                <button type="button">
                  Копировать<img src="@/assets/img/icon-copy.svg" alt="svg" />
                </button>
              </div>
            </div>
          </div>
          <div class="row mb-md-5">
            <div class="col-12 col-xl-4 mb-5 mb-lg-0 slider-offer-wrapper">
              <div class="slider-offer anketa-slider mb-5">
                <div
                  v-if="girlAnketa.photo.length"
                  class="anketa-slider__big mb-0 mb-md-5">
                  <swiper
                    :loop="true"
                    :pagination="{ clickable: true }"
                    :slidesPerView="1"
                    :navigation="{
                      nextEl: '.next',
                      prevEl: '.prev',
                    }"
                    :thumbs="{ swiper: thumbsSwiper }"
                    :modules="modules"
                    :mousewheel="false"
                    :grabCursor="false"
                    :effect="'fade'"
                    class="mySwiper2">
                    <swiper-slide
                      v-for="(item, i) in girlAnketa.photo"
                      :key="i">
                      <Fancybox
                        :options="optionsFancyBox"
                        :style="{
                          width: '100%',
                          height: '100%',
                        }">
                        <a
                          data-fancybox="FancyBoxGallery"
                          class="slide-offer"
                          :href="this.apiDomine + '/web/uploads' + item.pic"
                          :style="{
                            display: 'block',
                            width: '100%',
                            height: '100%',
                            backgroundSize: 'cover',
                            backgroundRepeat: 'no-repeat',
                            backgroundPosition: 'center',
                            backgroundImage:
                              'url(' +
                              this.apiDomine +
                              '/web/uploads' +
                              item.pic +
                              ')',
                          }">
                        </a>

                        <a
                          v-for="(itemHide, index) in girlAnketa.photo.filter(
                            (element) => element.pic !== item.pic
                          )"
                          :key="index"
                          data-fancybox="FancyBoxGallery"
                          class="slide-offer"
                          :href="
                            itemHide.pic !== item.pic
                              ? this.apiDomine + '/web/uploads' + itemHide.pic
                              : ''
                          "
                          :style="{
                            display: 'none',
                          }"
                          >{{ this.apiDomine }}
                        </a>
                      </Fancybox>
                    </swiper-slide>
                    <div class="prev">
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        width="16"
                        height="16"
                        fill="currentColor"
                        class="bi bi-chevron-left"
                        viewBox="0 0 16 16">
                        <path
                          fill-rule="evenodd"
                          d="M11.354 1.646a.5.5 0 0 1 0 .708L5.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0z" />
                      </svg>
                    </div>
                    <div class="next">
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        width="16"
                        height="16"
                        fill="currentColor"
                        class="bi bi-chevron-right"
                        viewBox="0 0 16 16">
                        <path
                          fill-rule="evenodd"
                          d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708z" />
                      </svg>
                    </div>
                  </swiper>
                  <div class="anketa-slider__small">
                    <swiper
                      :loop="true"
                      @swiper="setThumbsSwiper"
                      :slidesPerView="4"
                      :breakpoints="swiperOptions.breakpoints"
                      :freeMode="true"
                      :spaceBetween="20"
                      :mousewheel="true"
                      :watchSlidesProgress="true"
                      :modules="modules"
                      class="mySwiper slider-mini">
                      <swiper-slide
                        class="mt-2"
                        v-for="(item, i) in girlAnketa.photo"
                        :key="i">
                        <div class="slider-mini__img">
                          <img
                            :src="this.apiDomine + '/web/uploads' + item.pic"
                            alt="girls"
                            loading="lazy" />
                        </div>
                      </swiper-slide>
                    </swiper>

                    <div v-if="user.role.item_name === 'admin'">
                      <button
                        @click="openModal = true"
                        class="video-test btn-gradient hover-btn">
                        Проверочное видео
                      </button>
                    </div>
                  </div>
                </div>
              </div>

              <teleport to="body">
                <div v-if="openModal" class="modal__window-wrapper">
                  <div
                    class="modal__window bg-white"
                    :class="
                      !positionModal ? 'right-position' : 'left-position'
                    ">
                    <div class="modal__navigation">
                      <div class="btns-postions">
                        <button @click="positionModal = true" class="left_arr">
                          <i class="bi bi-arrow-left"></i>
                        </button>
                        <button
                          @click="positionModal = false"
                          class="right_arr">
                          <i class="bi bi-arrow-right"></i>
                        </button>
                        <button @click="openModal = false" class="close">
                          <i class="bi bi-x-circle"></i>
                        </button>
                      </div>
                    </div>
                    <div class="modal__video-block">
                      <video
                        class="modal__video-m"
                        autoplay
                        controls
                        playsinline
                        webkit-playsinline
                        preload="metadata">
                        <source
                          :src="
                            this.apiDomine +
                            '/web/uploads/girls/verificationVideo/' +
                            this.girlAnketa.verification_video
                          " />
                      </video>
                    </div>
                    <div class="modal__btn-succesfull d-flex gap-3">
                      <button
                        @click="confirmVideo"
                        class="btn-gradient hover-btn">
                        Подтвердить
                      </button>
                      <button
                        @click="rejectVideo"
                        class="btn-gradient hover-btn">
                        Отклонить
                      </button>
                    </div>
                  </div>
                </div>
              </teleport>
              <div
                v-if="girlAnketa.video !== null"
                class="mt-5 d-none d-xl-block">
                <div class="anketa-title mt-5 pt-3 pt-sm-5 mb-3">
                  Видеовизитка
                </div>
                <div class="anketa-card__video">
                  <VideoIndividually
                    :videoCards="girlAnketa.video"
                    :name="girlAnketa.name"
                    :photos="girlAnketa.photo"
                    :heightFixed="true" />
                </div>
              </div>
            </div>
            <div class="col-12 col-xl-8">
              <div class="row anketa-descr">
                <div class="col-12">
                  <div class="row align-items-center">
                    <div class="">
                      <div class="anketa-descr__name name__cont">
                        <div class="anketa-name anketa__name mb-2 mb-sm-0">
                          {{ girlAnketa.name }}, {{ m_age(girlAnketa.age) }}
                          <div class="status ms-3">
                            <span
                              :class="[
                                'circle-m',
                                activOrOff ? 'online' : 'offline',
                              ]"></span>
                          </div>
                        </div>
                        <div class="d-flex">
                          <div
                            class="vk-like-button"
                            :class="{ liked: like === '#4a76a8' }"
                            role="button"
                            @click="addLike">
                            <svg
                              class="vk-like-icon"
                              xmlns="http://www.w3.org/2000/svg"
                              viewBox="0 0 20 20"
                              width="18"
                              height="18">
                              <path
                                :fill="isLiked ? '#4a76a8' : '#a3a3a3'"
                                d="M10 18s-6.667-4.4-8.333-8.167C.508 7.279 1.944 4 5.167 4c1.528 0 3.03.95 3.833 2.111C9.803 4.95 11.305 4 12.833 4 16.056 4 17.492 7.279 16.333 9.833 14.667 13.6 10 18 10 18z" />
                            </svg>
                            <span class="like-count">{{
                              girlAnketa.stat?.like
                            }}</span>
                          </div>
                          <div class="copy__url" role="button" @click="copyUrl">
                            <div class="">
                              <img src="@/assets/img/reply.png" alt="" />
                            </div>
                            <div class="copy-alert" ref="copyAlert"></div>
                          </div>
                          <div
                            class="col-auto me-2 stat-grey d-flex align-items-center p-1 pe-2">
                            <div class="ms-2 me-2 d-flex align-items-center">
                              <img src="@/assets/img/eye-grey.png" alt="" />
                            </div>
                            <span class="sm-text">{{
                              girlAnketa.stat?.view
                            }}</span>
                          </div>
                          <div
                            v-if="false"
                            class="col-auto me-2 stat-grey d-flex align-items-center p-1"
                            role="button"
                            style="width: 33px">
                            <div class="mx-2 d-flex align-items-center">
                              <img src="@/assets/img/stroke.png" alt="" />
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>

                    <div
                      v-show="lastActivity"
                      class="col-12 activity__anket anketa-text mb-2">
                      была: {{ lastActivity }}
                    </div>
                    <div class="contacts-wrapper">
                      <div v-if="girlAnketa.tel" class="contact-item">
                        <button
                          class="contact-btn"
                          @click="showNumber = !showNumber">
                          {{
                            showNumber ? formatPhoneNumber : "Показать номер"
                          }}
                        </button>
                      </div>

                      <div v-if="girlAnketa.whatsapp" class="contact-item">
                        <a
                          :href="`https://wa.me/${girlAnketa.whatsapp}`"
                          target="_blank"
                          class="contact-btn outline"
                          title="WhatsApp">
                        </a>
                      </div>

                      <div v-if="girlAnketa.tg" class="contact-item">
                        <a
                          :href="`https://t.me/${girlAnketa.tg}`"
                          target="_blank"
                          class="contact-btn outline"
                          title="Telegram">
                        </a>
                      </div>

                      <div
                        v-if="+user.user_id !== +girlAnketa.user_id"
                        class="contact-item">
                        <button
                          class="contact-btn outline"
                          @click="sendMessageClick">
                          Написать
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>



              <div class="anketa-descr__about-title my-2">
                Моё местоположение
              </div>
              <div class="anketa-descr__about">
                <div
                  v-if="girlAnketa.city?.name"
                  class="anketa-descr__about-row">
                  <div class="anketa-descr__about-left">Город:</div>
                  <div class="anketa-descr__about-strip"></div>
                  <div class="anketa-descr__about-info">
                    {{ girlAnketa.city?.name }}
                  </div>
                </div>
                <div
                  v-if="girlAnketa.metro?.name"
                  class="anketa-descr__about-row">
                  <div class="anketa-descr__about-left">Метро:</div>
                  <div class="anketa-descr__about-strip"></div>
                  <div class="anketa-descr__about-info">
                    {{ girlAnketa.metro?.name }}
                  </div>
                </div>
                <div v-if="girlDistrict" class="anketa-descr__about-row">
                  <div class="anketa-descr__about-left">Район:</div>
                  <div class="anketa-descr__about-strip"></div>
                  <div class="anketa-descr__about-info">{{ girlDistrict }}</div>
                </div>
              </div>
              <div>
                <div class="about-grid">


                  <div v-if="girlAnketa.breast" class="grid-row">
                    <div class="label">Размер груди:</div>
                    <div class="value">{{ girlAnketa.breast }}</div>
                  </div>

                  <div v-if="girlAnketa.height" class="grid-row">
                    <div class="label">Рост:</div>
                    <div class="value">{{ girlAnketa.height }} см</div>
                  </div>

                  <div v-if="girlAnketa.weight" class="grid-row">
                    <div class="label">Вес:</div>
                    <div class="value">{{ girlAnketa.weight }} кг</div>
                  </div>

                  <div v-if="girlAnketa.clothing_size" class="grid-row">
                    <div class="label">Размер одежды:</div>
                    <div class="value">{{ girlAnketa.clothing_size }}</div>
                  </div>

                  <div v-if="girlAnketa.shoe_size" class="grid-row">
                    <div class="label">Размер обуви:</div>
                    <div class="value">{{ girlAnketa.shoe_size }}</div>
                  </div>

                  <div v-if="girlAnketa.intim_haircut" class="grid-row">
                    <div class="label">Интимная стрижка:</div>
                    <div class="value">{{ girlAnketa.intim_haircut }}</div>
                  </div>

                  <div v-if="girlAnketa.hair_color" class="grid-row">
                    <div class="label">Цвет волос:</div>
                    <div class="value">{{ girlAnketa.hair_color }}</div>
                  </div>

                  <div v-if="girlAnketa.appearance" class="grid-row">
                    <div class="label">Внешность:</div>
                    <div class="value">{{ girlAnketa.appearance }}</div>
                  </div>

                  <div v-if="girlAnketa.nationality" class="grid-row">
                    <div class="label">Национальность:</div>
                    <div class="value">{{ girlAnketa.nationality }}</div>
                  </div>

                  <div v-if="girlAnketa.body_type" class="grid-row">
                    <div class="label">Телосложение:</div>
                    <div class="value">{{ girlAnketa.body_type }}</div>
                  </div>
                  <div v-if="girlAnketa.not_younger" class="grid-row">
                    <div class="label">Не моложе:</div>
                    <div class="value">{{ girlAnketa.not_younger }}</div>
                  </div>
                  <div v-if="girlAnketa.not_older" class="grid-row">
                    <div class="label">Не старше:</div>
                    <div class="value">{{ girlAnketa.not_older }}</div>
                  </div>
                </div>
              </div>
              <div
                v-show="girlAnketa.description"
                class="anketa-descr__about-title my-2">
                Моё сообщение тебе
              </div>
               <div
                v-if="girlAnketa.description"
                class="anketa-descr__text my-3">
                {{ girlAnketa.description }}
              </div>

              <div v-if="girlAnketa.video !== null" class="d-block d-xl-none">
                <div class="anketa-title pt-3 pt-sm-5 mb-3">Видеовизитка</div>
                <div class="anketa-card__video">
                  <VideoIndividually
                    :videoCards="girlAnketa.video"
                    :name="girlAnketa.name"
                    :photos="girlAnketa.photo" />
                </div>
              </div>

              <!--                    Тарифы-->
              <div class="row mt-4">
                <div class="col-12">
                  <div class="anketa-descr__about-title my-2 mt-3">Тариф</div>
                  <div class="row pe-1 ps-1 ps-md-0 pe-md-0">
                    <div class="col-6 col-md-3 p-1">
                      <div class="card card-day h-100">
                        <div class="card-body">
                          <div class="row mt-4">
                            <div class="col-6">
                              <img
                                src="@/assets/img/day1.png"
                                alt="day_1"
                                class="img-fluid" />
                            </div>
                            <div class="col-6 card-day_title">1 час</div>
                          </div>
                          <div class="row mt-4 align-items-center">
                            <div class="col-4 card-day_text">У меня</div>
                            <div class="col-8 card-day_price">
                              {{ girlAnketa.price_hour_app || 0 }}
                              {{ girlAnketa.currency?.symbol }}
                            </div>
                          </div>
                          <div class="row mt-1 mb-2">
                            <div class="col-5 card-day_text">У тебя</div>
                            <div class="col-7 card-day_price">
                              {{ girlAnketa.price_hour_exit || 0 }}
                              {{ girlAnketa.currency?.symbol }}
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                    <div class="col-6 col-md-3 p-1">
                      <div class="card card-day h-100">
                        <div class="card-body">
                          <div class="row mt-4">
                            <div class="col-6">
                              <img src="@/assets/img/day2.png" alt="day_2" />
                            </div>
                            <div class="col-6 card-day_title">2 часа</div>
                          </div>
                          <div class="row mt-4 align-items-center">
                            <div class="col-4 card-day_text">У меня</div>
                            <div class="col-8 card-day_price">
                              {{ girlAnketa.price_two_hour_app || 0 }}
                              {{ girlAnketa.currency?.symbol }}
                            </div>
                          </div>
                          <div class="row mt-1 mb-2">
                            <div class="col-5 card-day_text">У тебя</div>
                            <div class="col-7 card-day_price">
                              {{ girlAnketa.price_two_hour_exit || 0 }}
                              {{ girlAnketa.currency?.symbol }}
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                    <div class="col-6 col-md-3 p-1">
                      <div class="card card-night h-100">
                        <div class="card-body">
                          <div class="row mt-3">
                            <div class="col-6">
                              <img
                                src="@/assets/img/night1.png"
                                alt="night_1" />
                            </div>
                            <div class="col-6 card-night_title">1 час</div>
                          </div>
                          <div class="row mt-3 align-items-center">
                            <div class="col-4 card-night_text">У меня</div>
                            <div class="col-8 card-night_price">
                              {{ girlAnketa.price_hour_night_app || 0 }}
                              {{ girlAnketa.currency?.symbol }}
                            </div>
                          </div>
                          <div class="row mt-1 mb-2">
                            <div class="col-5 card-night_text">У тебя</div>
                            <div class="col-7 card-night_price">
                              {{ girlAnketa.price_hour_night_exit || 0 }}
                              {{ girlAnketa.currency?.symbol }}
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                    <div class="col-6 col-md-3 p-1">
                      <div class="card card-night h-100">
                        <div class="card-body">
                          <div class="row mt-3">
                            <div class="col-6">
                              <img
                                src="@/assets/img/night2.png"
                                alt="night_2"
                                height="61" />
                            </div>
                            <div class="col-6 card-night_title">Ночь</div>
                          </div>
                          <div class="row mt-3 align-items-center">
                            <div class="col-4 card-night_text">У меня</div>
                            <div class="col-8 card-night_price">
                              {{ girlAnketa.price_night_app || 0 }}
                              {{ girlAnketa.currency?.symbol }}
                            </div>
                          </div>
                          <div class="row mt-1 mb-2">
                            <div class="col-5 card-night_text">У тебя</div>
                            <div class="col-7 card-night_price">
                              {{ girlAnketa.price_night_exit || 0 }}
                              {{ girlAnketa.currency?.symbol }}
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <!--                    Теги-->
              <div
                v-if="girlAnketa.tags && girlAnketa.tags.length"
                class="row mt-4">
                <div class="col-12">
                  <div class="anketa-descr__about-title my-2 mt-3">Теги</div>
                </div>
                <div class="row mt-3">
                  <router-link
                    :to="`/compilation-tag/${item.tag.id}`"
                    class="col-auto p-1 text-decoration-none"
                    v-for="item in girlAnketa.tags"
                    :key="item.id">
                    <div class="tag-item p-2">
                      <img
                        :src="this.apiDomine + '/web/uploads/' + item.tag.pic"
                        alt="tag"
                        class="tag-img" />
                      <span class="p-3">{{ item.tag.name }}</span>
                    </div>
                  </router-link>
                </div>
              </div>
            </div>
          </div>
          <hr />
          <!--Service-->
          <div class="row pt-md-3">
            <div class="col-12">
              <div class="anketa-descr__about-title my-2 mt-3">
                Предпочтения
              </div>
            </div>
            <div class="col-12">
              <div class="row justify-content-center">
                <div class="col-12 col-sm-10">
                  <ul
                    class="nav nav-pills mb-3 mt-3 justify-content-start"
                    id="pills-tab"
                    role="tablist">
                    <li
                      class="nav-item"
                      role="presentation"
                      v-if="this.service.sex.length != 0">
                      <button
                        class="nav-link hover-btn"
                        :class="{ active: activeTab === 'sex' }"
                        id="pills-sex-tab"
                        data-bs-toggle="pill"
                        data-bs-target="#pills-sex"
                        type="button"
                        role="tab"
                        aria-controls="pills-sex"
                        aria-selected="true">
                        Секс
                      </button>
                    </li>
                    <li
                      class="nav-item"
                      role="presentation"
                      v-if="this.service.massage.length != 0">
                      <button
                        class="nav-link hover-btn"
                        :class="{ active: activeTab === 'massage' }"
                        id="pills-massage-tab"
                        data-bs-toggle="pill"
                        data-bs-target="#pills-massage"
                        type="button"
                        role="tab"
                        aria-controls="pills-massage"
                        aria-selected="false">
                        Массаж
                      </button>
                    </li>
                    <li
                      class="nav-item"
                      role="presentation"
                      v-if="this.service.strip.length != 0">
                      <button
                        class="nav-link hover-btn"
                        :class="{ active: activeTab === 'strip' }"
                        id="pills-strip-tab"
                        data-bs-toggle="pill"
                        data-bs-target="#pills-strip"
                        type="button"
                        role="tab"
                        aria-controls="pills-strip"
                        aria-selected="false">
                        Стриптиз
                      </button>
                    </li>
                    <li
                      class="nav-item"
                      role="presentation"
                      v-if="this.service.extreme.length != 0">
                      <button
                        class="nav-link hover-btn"
                        :class="{ active: activeTab === 'extreme' }"
                        id="pills-extreme-tab"
                        data-bs-toggle="pill"
                        data-bs-target="#pills-extreme"
                        type="button"
                        role="tab"
                        aria-controls="pills-extreme"
                        aria-selected="false">
                        Экстрим
                      </button>
                    </li>
                    <li
                      class="nav-item"
                      role="presentation"
                      v-if="this.service.bdsm.length != 0">
                      <button
                        class="nav-link hover-btn"
                        :class="{ active: activeTab === 'dbsm' }"
                        id="pills-bdsm-tab"
                        data-bs-toggle="pill"
                        data-bs-target="#pills-bdsm"
                        type="button"
                        role="tab"
                        aria-controls="pills-bdsm"
                        aria-selected="false">
                        Садо-мазо
                      </button>
                    </li>
                    <li
                      class="nav-item"
                      role="presentation"
                      v-if="this.service.dop.length != 0">
                      <button
                        class="nav-link hover-btn"
                        :class="{ active: activeTab === 'dop' }"
                        id="pills-dop-tab"
                        data-bs-toggle="pill"
                        data-bs-target="#pills-dop"
                        type="button"
                        role="tab"
                        aria-controls="pills-dop"
                        aria-selected="false">
                        Разное
                      </button>
                    </li>
                  </ul>
                </div>
              </div>
              <div class="tab-content" id="pills-tabContent">
                <div
                  class="tab-pane fade"
                  :class="{ 'show active': activeTab === 'sex' }"
                  id="pills-sex"
                  role="tabpanel"
                  aria-labelledby="pills-sex-tab"
                  tabindex="0">
                  <div class="row">
                    <div
                      class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2"
                      v-for="item in service.sex"
                      :key="item.id">
                      <div class="row">
                        <div class="col-12 d-flex align-items-center">
                          <img src="@/assets/img/check-green.svg" alt="" />
                          <span class="name-service fw-bold">{{
                            item.service.name
                          }}</span>
                        </div>
                        <div v-if="item.description" class="col-12 mt-2">
                          <span class="desc-service ps-3">{{
                            item.description
                          }}</span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <div
                  class="tab-pane fade"
                  :class="{ 'show active': activeTab === 'massage' }"
                  id="pills-massage"
                  role="tabpanel"
                  aria-labelledby="pills-massage-tab"
                  tabindex="0">
                  <div class="row">
                    <div
                      class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2"
                      v-for="item in service.massage"
                      :key="item.id">
                      <div class="row">
                        <div class="col-12 d-flex align-items-center">
                          <img src="@/assets/img/check-green.svg" alt="" />
                          <span class="name-service fw-bold">{{
                            item.service.name
                          }}</span>
                        </div>
                        <div v-if="item.description" class="col-12 mt-2">
                          <span class="desc-service ps-3">{{
                            item.description
                          }}</span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <div
                  class="tab-pane fade"
                  :class="{ 'show active': activeTab === 'strip' }"
                  id="pills-strip"
                  role="tabpanel"
                  aria-labelledby="pills-strip-tab"
                  tabindex="0">
                  <div class="row">
                    <div
                      class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2"
                      v-for="item in service.strip"
                      :key="item.id">
                      <div class="row">
                        <div class="col-12 d-flex align-items-center">
                          <img src="@/assets/img/check-green.svg" alt="" />
                          <span class="name-service fw-bold">{{
                            item.service.name
                          }}</span>
                        </div>
                        <div v-if="item.description" class="col-12 mt-2">
                          <span class="desc-service ps-3">{{
                            item.description
                          }}</span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <div
                  class="tab-pane fade"
                  :class="{ 'show active': activeTab === 'extreme' }"
                  id="pills-extreme"
                  role="tabpanel"
                  aria-labelledby="pills-extreme-tab"
                  tabindex="0">
                  <div class="row">
                    <div
                      class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2"
                      v-for="item in service.extreme"
                      :key="item.id">
                      <div class="row">
                        <div class="col-12 d-flex align-items-center">
                          <img src="@/assets/img/check-green.svg" alt="" />
                          <span class="name-service fw-bold">{{
                            item.service.name
                          }}</span>
                        </div>
                        <div v-if="item.description" class="col-12 mt-2">
                          <span class="desc-service ps-3">{{
                            item.description
                          }}</span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <div
                  class="tab-pane fade"
                  :class="{ 'show active': activeTab === 'bdsm' }"
                  id="pills-bdsm"
                  role="tabpanel"
                  aria-labelledby="pills-bdsm-tab"
                  tabindex="0">
                  <div class="row">
                    <div
                      class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2"
                      v-for="item in service.bdsm"
                      :key="item.id">
                      <div class="row">
                        <div class="col-12 d-flex align-items-center">
                          <img src="@/assets/img/check-green.svg" alt="" />
                          <span class="name-service fw-bold">{{
                            item.service.name
                          }}</span>
                        </div>
                        <div v-if="item.description" class="col-12 mt-2">
                          <span class="desc-service ps-3">{{
                            item.description
                          }}</span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <div
                  class="tab-pane fade"
                  :class="{ 'show active': activeTab === 'dop' }"
                  id="pills-dop"
                  role="tabpanel"
                  aria-labelledby="pills-dop-tab"
                  tabindex="0">
                  <div class="row">
                    <div
                      class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2"
                      v-for="item in service.dop"
                      :key="item.id">
                      <div class="row">
                        <div class="col-12 d-flex align-items-center">
                          <img src="@/assets/img/check-green.svg" alt="" />
                          <span class="name-service fw-bold">{{
                            item.service.name
                          }}</span>
                        </div>
                        <div v-if="item.description" class="col-12 mt-2">
                          <span class="desc-service ps-3">{{
                            item.description
                          }}</span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="row anketa-contact my-5">
            <hr />
            <!-- TODO: Закомментированный ниже блок вешает страницу!-->
            <!--                <div class="col-12 col-lg-6">
                    <div class="anketa-video">
                        <div class="anketa-chat" v-if="this.user.isLogged">
                            <div class="anketa-contact__title anketa-title mb-2">Общайся со&nbsp;мной онлайн</div>
                            <Chat/>
                        </div>
                        &lt;!&ndash; <div class="anketa-video__top d-flex justify-content-between">
                            <div class="anketa-contact__title anketa-title mb-2">Вебкам</div>
                            <div class="anketa-contact__text"><span><img src="@/assets/img/selection2.svg" alt="svg"></span>Сейчас онлайн</div>
                        </div> &ndash;&gt;
                        <div class="anketa-video__img anketa-contact__img">
                            &lt;!&ndash; <img src="@/assets/img/video-player.webp" alt="..."> &ndash;&gt;
                            <button class="mt-3 anketa-video__btn">Пригласить в приват</button>
                        </div>
                        <div class="anketa-present">
                            <div class="anketa-present__title">Радуйте модель чаевыми</div>
                            <div class="anketa-present__sum">
                                <button v-for="(item,i) in present" :key="i" @click="currencyBtn(item.sum)">{{
                                        item.sum
                                    }}{{ girlAnketa.currency?.symbol }}
                                </button>
                            </div>

                            <div class="anketa-present-bott row align-items-center">
                                <div class="col-12 col-lg-6">
                                    <div class="anketa-present-bott__choice">
                                        <div class="anketa-present-bott__text">Введите сумму чаевых</div>
                                        <label class="anketa-present-bott__label d-flex">
                                            <input type="text" class="anketa-present-bott__input" v-on:keyup="setTel"
                                                   v-model="currency">
                                            {{ girlAnketa.currency?.symbol }}
                                        </label>
                                    </div>

                                </div>
                                <div class="col-12 col-lg-5 mb-3 mb-lg-0">
                                    <div class="anketa-present-bott__btns row">
                                        <div class="col-6 py-2">
                                            <button>Отправить</button>
                                        </div>
                                        <div class="col-6 py-2">
                                            <button>Пополнить баланс</button>
                                        </div>
                                    </div>
                                </div>


                            </div>
                        </div>
                    </div>
                </div> -->

            <div class="col-12">
              <div class="anketa-mess">
                <!-- <div class="anketa-contact__title anketa-title mb-2">Общайся со&nbsp;мной онлайн</div> -->
                <div class="anketa-raiting">
                  <feedback :id="girlAnketa?.id"></feedback>
                </div>
              </div>
            </div>
          </div>
          <div class="row">
            <MyLife :girlAnketa="girlAnketa" />
          </div>
        </div>
        <all-tags class="pb-5" />
      </FormBase>
    </div>
  </div>

  <log-in-card
    :show="this.show"
    @loginSuccess="hideDialog"
    @hideDialog="hideDialog"
    :title="loginTitle" />
</template>
<style scoped lang="scss">
.anketa__name {
  display: flex;
  align-items: center;
}
.name__cont {
  display: flex;
  justify-content: space-between;
  gap: 20px;
}
.sm-text {
  font-size: 14px;
}
.vk-like-button {
  display: inline-flex;
  align-items: center;
  padding: 4px 8px;
  border-radius: 16px;
  background-color: #f2f3f5;
  color: #333;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s ease;

  .like-icon {
    width: 18px;
    height: 18px;
    margin-right: 6px;
    transition: transform 0.2s ease;
  }

  &:hover {
    background-color: #e0e0e0;

    .like-icon {
      transform: scale(1.1);
    }
  }

  &.liked {
    background-color: #e1ecf7;

    .like-count {
      color: #4a76a8;
      font-weight: 600;
    }
  }
}

.vk-like-button {
  display: inline-flex;
  align-items: center;
  padding: 4px 8px;
  border-radius: 16px;
  background-color: #f2f3f5;
  cursor: pointer;
  transition: background-color 0.2s ease;

  &:hover {
    background-color: #e0e0e0;
  }

  &.liked {
    background-color: #e1ecf7;

    .like-count {
      color: #4a76a8;
      font-weight: 600;
    }
  }

  .vk-like-icon {
    margin-right: 6px;
    transition: transform 0.2s ease;
  }

  &:hover .vk-like-icon {
    transform: scale(1.1);
  }

  .like-count {
    font-size: 14px;
    color: #333;
  }
}

.copy__url {
  background: #f2f3f5;
  padding: 10px;
  div {
  }
}
.activity__anket {
  color: #555;
}

.contacts-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  align-items: center;
  margin-top: 12px;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 6px;
}

.contact-btn {
  background: transparent;
  border: none;
  color: #000;
  font-size: 14px;
  cursor: pointer;
  padding: 6px 12px;
  border-radius: 8px;
  transition: 0.3s ease;
}

.contact-btn.outline {
  border: 1px solid #999;
}

.contact-btn:hover {
  background-color: #f4f4f4;
}
.section-title {
  font-size: 18px;
  font-weight: 600;
  margin-top: 24px;
  margin-bottom: 12px;
  color: #222;
}

.about-grid {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 16px;
  background-color: #f9f9f9;
  border-radius: 12px;
}

.grid-row {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  gap: 16px;
  border-bottom: 1px solid #eee;
  padding-bottom: 6px;
}

.label {
  font-weight: 500;
  color: #444;
  max-width: 45%;
  flex: 1 1 auto;
}

.value {
  color: #000;
  font-weight: 400;
  text-align: right;
  max-width: 50%;
  flex-shrink: 0;
}

/* Адаптив для мобилок */
@media (max-width: 600px) {
  .grid-row {
    flex-direction: column;
    align-items: flex-start;
    padding-bottom: 8px;
  }

  .label,
  .value {
    max-width: 100%;
    text-align: left;
  }
}


.about-grid {
  display: grid;
  grid-template-columns: 1fr 1fr; /* две равные колонки */
  gap: 12px 24px; /* расстояние между строками и колонками */
}

.grid-row {
  display: flex;
  gap: 6px;
}

.label {
  font-weight: 500;
  white-space: nowrap;
  color: #888; /* светло-серый для подписей */
  font-size: 14px;
}

.value {
  flex: 1;
  color: #555; /* белый или любой основной цвет текста */
  font-weight: 600;
  font-size: 15px;
}

.main-container {
  max-width: 1280px;
  margin: 0 auto;
}
.bg-main {
  background: #f6f7f8;
  padding-top: 60px;
  width: 100%;
  height: 100%;
}
.status {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
  font-size: 1rem;
}

.circle-m {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  display: inline-block;
}

.online {
  background-color: #4caf50; // зелёный
}

.offline {
  background-color: #f44336; // красный
}
.modal {
  &__window-wrapper {
    position: fixed;
    background: transparent;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
    display: flex;
    z-index: 10;

    video {
      height: 600px;
      max-width: 600px;
      width: 100%;
    }
  }

  &__window {
    position: absolute;
    top: 100px;
    border: 1px solid #979797;
    padding: 10px 10px;
    border-radius: 5px;
  }

  &__navigation {
    display: flex;
    align-items: center;
  }
}

.right-position {
  right: 15px !important;
}

.left-position {
  left: 15px !important;
}

.anketa {
  user-select: none;
}

.anketa-descr__name {
  @media screen and (max-width: 576px) {
    justify-content: flex-start !important;
    flex-wrap: wrap;

    .anketa-name {
      width: 100%;
    }
  }
}

@keyframes copy-alert-hide {
  0% {
    opacity: 1;
    visibility: visible;
  }

  60% {
    opacity: 1;
    visibility: visible;
  }

  100% {
    opacity: 0;
    visibility: hidden;
  }
}

.mySwiper2 .prev,
.mySwiper2 .next {
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  position: absolute;
  top: calc(50% - 15px);
  background: #fff;
  opacity: 0.4;
  width: 29px;
  height: 29px;
  border: 1px solid #bdc1d1;
  border-radius: 5px;
  z-index: 9;
}

.mySwiper2 .prev {
  left: 15px;
}

.mySwiper2 .next {
  right: 15px;
}

.mySwiper2 .swiper-slide {
  opacity: 1;
  border: 1px solid #39354b;
  border-radius: 8px;
}

.swiper-slide-thumb-active .slider-mini__img {
  border: 3px solid transparent;
  border-image: linear-gradient(to right, #21045d, #db0f00);
  -moz-border-image: -moz-linear-gradient(left, #21045d, #db0f00);
  -webkit-border-image: -webkit-linear-gradient(left, #21045d, #db0f00);
  border-image-slice: 1;
}

.work {
  background: #22bc32;
  border-radius: 46px;
  font-size: 12px;
  color: #fff;
  font-weight: 600;
  line-height: 23px;
  padding: 0px 10px;
}

.work.off {
  background: #db0f00;
}

.like {
  background: rgba(255, 64, 50, 0.1);
  border-radius: 40px;
  color: #ff4032;
}

.like div {
  background: #ff4032;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  opacity: 0.4;
  border-radius: 50%;
  color: #fff;
  width: 20px;
  height: 20px;
}

.stat-grey {
  background: rgba(82, 86, 101, 0.7);
  border-radius: 40px;
  color: #fff;
}

.anketa-id {
  color: #525665;
  font-size: 10px;
}

.anketa-text {
  color: #525665;
  font-size: 14px;
}

.btn-gradient {
  border-radius: 30px;
  background: linear-gradient(
    93deg,
    #72666a 0%,
    #524b5f 50.09%,
    #201f36 99.15%
  );
  color: #fff;
  font-weight: 600;
  padding: 5px 10px;
}

.btn-gradient:hover {
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.9);
}

.card-body {
  text-shadow: 2px 2px 5px #0000007b;
}

.card-day {
  background: url("@/assets/img/condom-day.png");
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center;
  border: none;

  &_title {
    font-weight: 700;
    font-size: 18px;
    color: #201f36;
    display: flex;
    align-items: center;
    justify-content: flex-end;
  }

  &_text {
    color: rgba(72, 76, 94, 0.6);
    font-size: 13px;
    padding-right: 0;
  }

  &_price {
    font-weight: 700;
    font-size: 16px;
    color: #201f36;
    text-align: end;
  }
}

.card-night {
  background: url("@/assets/img/condom-night.png");
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center;
  border: none;

  &_title {
    font-weight: 700;
    font-size: 18px;
    color: #fff;
    display: flex;
    align-items: center;
    justify-content: flex-end;
  }

  &_text {
    color: #fff;
    font-size: 13px;
    padding-right: 0;
  }

  &_price {
    font-weight: 700;
    font-size: 16px;
    color: #fff;
    text-align: end;
  }
}

//service

.tab-pane img {
  max-width: 17px;
  margin-right: 15px;
}

.tab-pane .desc-service {
  color: #22bc32;
}

.nav-link {
  background: #21232f;
  border-radius: 20px;
  border: none;
  color: #fff;
  padding: 5px 30px;
}

.nav-pills {
  gap: 20px;

  @media (max-width: 561px) {
    gap: 10px;
  }
}

.nav-pills .nav-link {
  background: #21232f;
  border-radius: 20px;
  border: none;
}

.nav-pills .nav-link:hover {
  color: #1ad42c;
}

.nav-pills .nav-link.active {
  background: #21232f;
  border-radius: 20px;
  border: none;
  color: #1ad42c;
}

.name-service {
  font-size: 14px;
  font-weight: 200;
  color: #525665;
}

.title-service {
  font-size: 24px;
  color: #525665;
}

//end service

////////////////

.copy-alert {
  position: absolute;
  left: 0;
  top: -110%;
  padding: 6px;
  padding-left: 0.5rem;
  font-size: 12px;
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
  background-color: #dedede;
  padding: 5px 10px;
  display: inline-block;
  border-radius: 5px;
  margin-top: 5px;
  width: max-content;

  &.success {
    color: #333;
    animation: copy-alert-hide 2s linear;
  }

  &.error {
    color: red;
    animation: copy-alert-hide 2s linear;
  }
}

.anketa {
  @media (max-width: 1023px) {
    padding-top: 50px;
  }
}

.anketa-top {
  &__url {
    position: relative;
    display: flex;
    align-items: center;
    margin-left: auto;
    border-radius: 12px;
    border: 1px solid #bdc1d1;
    background: #fff;
    height: 100%;

    @media (max-width: 991px) {
      border: none;
    }

    button {
      color: #fff;
      font-size: 10px;
      border-radius: 10px;
      background: #39354b;
      display: inline-block;
      height: 35px;
      line-height: 35px;
      width: 100px;
      text-align: center;
    }

    &--mobile {
      button {
        color: #fff;
        font-size: 7px;
        font-weight: 700;
        height: auto;
        width: auto;
        line-height: 100%;
        white-space: nowrap;
        padding: 3px 7px;
        border-radius: 4px;

        img {
          padding-left: 5px;
          max-width: 15px;
        }
      }
    }
  }
}

.slider-offer-wrapper {
  @media (max-width: 1199px) {
    margin-bottom: 6rem !important;
  }
}

.anketa-slider {
  display: flex;
  max-width: 100% !important;

  @media (max-width: 1199px) {
    margin: 0 auto;
    justify-content: center;
  }

  .slider-offer {
    @media (max-width: 1199px) {
      margin: 0 auto;
      justify-content: center;
    }
  }

  &__big {
    width: 100%;
    max-width: 416px;
    height: 550px;
    margin-right: 10px;

    @media (max-width: 575px) {
      max-width: 350px;
      margin-right: 0;
    }
  }

  &__small {
    // height: 450px !important;

    @media (max-width: 575px) {
      height: auto !important;
    }

    .swiper-wrapper {
      display: flex;
      //flex-direction: column;
      justify-content: space-between;
      transform: translate3d(0px, 0px, 0px) !important;
    }

    .swiper-slide {
      border-radius: 8px;
      overflow: hidden;
      margin-bottom: 10px;
      margin-right: 7px !important;
      width: 73px !important;
      height: 73px !important;
      position: relative;

      @media (max-width: 1199px) {
        // width: 30px !important;
        // height: 30px !important;
        width: 65px !important;
        height: 65px !important;
      }
    }
  }
}

.anketa-more {
  &__arrow {
    display: flex;
    margin-left: 15px;

    span {
      display: flex;
      width: 9px;
      height: 9px;
      cursor: pointer;

      &:first-child {
        border-left: solid 2px #bdc1d1;
        border-bottom: solid 2px #bdc1d1;
        transform: rotate(45deg);
        margin-right: 10px;
      }

      &:last-child {
        border-right: solid 2px #39354b;
        border-bottom: solid 2px #39354b;
        transform: rotate(-45deg);
      }
    }
  }

  &__name {
    color: #484c5e;
    font-size: 16px;
    font-weight: 600;
    margin-left: 15px;
  }

  p {
    color: #525665;
    font-size: 14px;
    font-weight: 500;
  }
}

.anketa-descr {
  &__name {
    color: #2d2f3a;
    font-size: 32px;
    line-height: 100%;
    display: flex;
    align-items: center;

    @media (max-width: 991px) {
      font-size: 18px;
      justify-content: space-between;

      img {
        max-width: 15px;
        margin-bottom: 7px;
      }
    }

    span {
      cursor: pointer;

      // &:last-child {
      //   display: inline-block;
      //   width: 20px;
      //   height: 20px;
      //   position: relative;

      //   @media (max-width: 991px) {
      //     width: 15px;
      //     height: 15px;
      //     margin-bottom: 3px;
      //   }

      // &::after,
      // &:before {
      //   content: "";
      //   display: block;
      //   width: 100%;
      //   height: 1px;
      //   background-color: #2d2f3a;

      //   position: absolute;
      //   left: 0;
      //   top: 60%;
      // }

      // &:after {
      //   content: "";
      //   transform: rotate(45deg);
      // }

      // &::before {
      //   transform: rotate(-45deg);
      // }
      // }
    }
  }

  &__loock {
    color: #525665;
    font-size: 14px;
    font-weight: 700;
    line-height: 106.3%;
    display: flex;
    align-items: center;

    @media (max-width: 991px) {
      font-size: 12px;
    }
  }

  &__loock-counter {
    position: relative;
    color: #484c5e;
    font-size: 16px;
    display: flex;
    align-items: center;
    margin-left: 15px;

    &:before {
      content: "";
      display: block;
      background-image: url("@/assets/img/eye.png");
      background-repeat: no-repeat;
      background-size: cover;
      width: 20px;
      height: 20px;
      margin-right: 10px;
    }
  }

  &__active {
    display: flex;
    align-items: center;
    color: #525665;
    font-size: 14px;

    @media (max-width: 991px) {
      flex-wrap: wrap;
      font-size: 10px;
    }

    p {
      @media (max-width: 991px) {
        flex: 0 0 100%;
        max-width: 100%;
      }
    }
  }

  &__activ-date {
    display: flex;
    align-items: center;
    color: #484c5e;
    font-size: 16px;
    font-weight: 600;

    @media (max-width: 991px) {
      font-size: 12px;
    }

    span + span {
      margin-left: 10px;
    }
  }

  &__mess {
    button {
      color: #fff;
      background-color: #978b8a;
      font-size: 16px;
      padding: 10px 20px;
      border-radius: 32px;
      transition: 0.3s;

      @media (max-width: 991px) {
        font-size: 12px;
        padding: 3px 6px;
      }

      img {
        padding-left: 10px;

        @media (max-width: 991px) {
          padding-left: 2px;
          max-width: 12px;
        }
      }

      &:hover {
        box-shadow: 0 0 4px rgba(0, 0, 0, 0.5);
        background-color: lighten(#978b8a, 5%);
      }
    }
  }

  &__row {
    color: #525665;
    font-size: 14px;
    font-weight: 700;
    line-height: 106.3%;
    display: flex;
    align-items: center;

    span {
      background-color: #cdc8c7;
      border-radius: 6px;
      display: block;
      width: 30px;
      height: 30px;
      position: relative;
      margin-right: 10px;

      @media (max-width: 767px) {
        width: 20px;
        height: 20px;
      }

      img {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        margin: auto;

        @media (max-width: 767px) {
          width: 13px;
          height: 13px;
        }
      }
    }

    @media (max-width: 767px) {
      font-size: 10px;
    }
  }

  &__text {
    color: #312827;
    font-size: 14px;
    line-height: 120%;
  }

  &__about {
  }

  &__about-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  &__about-title {
    color: #2d2f3a;
    font-size: 24px;
    line-height: 100%;
  }

  &__about-left {
    color: #525665;
    font-size: 14px;
  }

  &__about-right {
    color: #312827;
    font-size: 14px;
  }

  &__about-strip {
    display: block;
    margin: 0 15px;
    height: 1px;
    background-color: rgba(#bdc1d14f, 0.31);
    flex-grow: 1;
  }
}

.anketa-title {
  color: #2d2f3a;
  font-size: 24px;
  font-weight: 600;
  line-height: 100%;
}

.anketa-contact {
  &__text {
    position: relative;
    color: #312827;
    font-size: 14px;
    display: flex;
    align-items: center;

    span {
      display: flex;
    }

    img {
      margin-right: 5px;
      margin-top: -2px;
    }
  }

  &__img {
    img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
  }
}

.anketa-video {
  &__btn {
    color: #fff;
    font-size: 10px;
    border-radius: 10px;
    background: #39354b;
    padding: 8px 18px;

    @media (max-width: 991px) {
      margin: 0 auto;
      display: block;
    }
  }
}

.anketa-present {
  margin-top: 55px;

  &__title {
    color: #39354b;
    font-size: 16px;
    font-weight: 700;
    margin: 10px 0;
  }

  &__sum {
    gap: 10px;
    display: flex;
    flex-wrap: wrap;

    button {
      color: #39354b;
      font-size: 14px;
      font-weight: 600;
      line-height: 100%;
      border-radius: 10px;
      background: #dce3ec;
      width: 62px;
      height: 30px;
      border: solid 1px transparent;
      transition: 0.3s;

      &:hover {
        border-color: #39354b;
      }
    }
  }
}

.anketa-present-bott {
  div {
    padding: 5px;

    @media (max-width: 991px) {
      padding: 0.2rem 0.5rem 0.5rem;
    }
  }

  &__btns {
    button {
      color: #fff;
      font-size: 10px;
      border-radius: 10px;
      background: #39354b;
      transition: 0.3s;
      width: 100%;
      display: block;
      padding: 10px;
      height: 100%;

      &:hover {
        background-color: lighten(#39354b, 10%);
      }
    }
  }

  &__text {
    color: #39354b;
    font-size: 16px;
  }

  &__label {
    border-radius: 10px;
    border: 1px solid #cfd7f2;
    padding: 5px 25px 5px 10px;
    max-width: 100px;
    display: block;
    position: relative;

    span {
      position: absolute;
      right: 5px;
      top: 50%;
      transform: translateY(-50%);
    }
  }

  &__input {
    border-bottom: solid 1px;
    color: #39354b;
    font-size: 14px;
    font-weight: 700;
    max-width: 100%;
  }

  &__choice {
    display: flex;
    align-items: center;
  }
}

.anketa-raiting {
  margin-top: 40px;

  &__row {
    display: flex;
  }

  &__info {
    position: relative;

    div {
      &:first-child {
        color: #2d2f3a;
        font-size: 24px;
        font-style: normal;

        span {
          color: #2d2f3a;
          font-size: 24px;
          font-weight: 700;
          line-height: 100%;
        }
      }

      &:last-child {
        color: #39354b;
        font-size: 16px;
        font-style: normal;
        position: absolute;
        white-space: nowrap;
      }
    }
  }

  &__star {
    margin-left: 5px;
    white-space: nowrap;

    img {
      width: 25px;
      height: 25px;
      margin-bottom: -7px;

      @media (max-width: 1400px) {
        width: 20px;
        height: 20px;
      }

      @media (max-width: 1200px) {
        width: 18px;
        height: 18px;
      }
    }
  }

  &__btns {
    display: flex;
    margin-left: auto;

    div + div {
      margin-left: 10px;
    }

    button {
      color: #fff;
      text-align: center;
      font-size: 10px;
      border-radius: 10px;
      background: #39354b;
      transition: 0.3s;
      padding: 8px 12px;
      display: flex;
      align-items: center;

      img {
        padding-right: 3px;
      }

      &:hover {
        background-color: lighten(#39354b, 10%);
      }
    }
  }
}

.anketa-raiting-graf {
  margin: 40px 0 20px;

  &__item {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  &__star {
    flex: 0 0 15%;
    max-width: 15%;
    white-space: nowrap;
    display: flex;
  }

  &__line {
    display: block;
    height: 4px;
    background-color: #bdc1d1;
    border-radius: 50px;
    flex: 0 0 70%;
    max-width: 70%;

    @media (max-width: 575px) {
      flex: 0 0 50%;
      max-width: 50%;
    }

    div {
      display: block;
      width: 80%;
      height: 100%;
      background-color: #484c5e;
      border-radius: 50px;
    }
  }

  &__result {
    flex: 0 0 5%;
    max-width: 5%;
    text-align: right;
  }
}

.anketa-recall {
  &__title {
    margin-bottom: 20px;
  }

  &__item {
    margin-bottom: 20px;

    &:last-child {
      margin-bottom: 0;
    }
  }

  &__name {
    color: #39354b;
    font-size: 19px;
    font-weight: 700;
    margin-bottom: 5px;
  }

  &__text {
    font-size: 14px;
    line-height: 120%;

    @media (max-width: 991px) {
      font-size: 12px;
    }
  }
}

.saved-heart {
  vertical-align: baseline;
}

.slide-offer {
  border-radius: 8px !important;
}

.tag-list {
  background: #201f36;
  color: #fff;
}

.tag-item {
  background-color: #2d2f3a;
  border-radius: 1.5rem;
  color: #fff;
  cursor: pointer;
  transition: all 1s ease;
}

.tag-img {
  width: 26px;
  height: 26px;
  border-radius: 50%;
}
</style>
