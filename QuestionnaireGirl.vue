<script>
import VueNextSelect from 'vue-next-select'
import "vue-next-select/dist/index.css";
import 'intl-tel-input/build/css/intlTelInput.css';
import 'intl-tel-input/build/js/intlTelInput.js';
import 'intl-tel-input/build/js/utils.js';
import axios from "axios";
import UiLoader from "@/components/ui/UiLoader.vue";
import intlTelInput from 'intl-tel-input';
import UploadVideo from "@/components/ui/uploadMedia/UploadVideo.vue";
import UploadTitlePhoto from "@/components/ui/uploadMedia/UploadTitlePhoto.vue";
import CardWhite from "@/components/ui/CardWhite.vue";
import UploadVideoCard from "@/components/ui/uploadMedia/UploadVideoCard.vue";
import TagsToProfile from "@/components/ui/common/tags/TagsToProfile.vue";
import TagAdd from "@/pages/Admin/AdminSettings/Lists/TagAdd.vue";
import UiSelect from "@/components/ui/UiSelect.vue";
import ModalDialog from "@/components/ui/common/ModalDialog.vue";
import CloseX from "@/components/ui/CloseX.vue";
import HintInput from "@/components/ui/hint/HintInput.vue";
import flashMassage from "@/mixins/flashMassage";
import {useCookies} from "vue3-cookies";
import UserSelect from "@/components/ui/hint/UserSelect.vue";

export default {
  components: {
    UserSelect,
    HintInput,
    CloseX, ModalDialog,
    UiSelect,
    TagAdd,
    TagsToProfile,
    UploadVideoCard,
    CardWhite,
    UploadTitlePhoto,
    UploadVideo,
    UiLoader,
    'vue-select': VueNextSelect,
    intlTelInput,
    flashMassage

  },
  mixins: [flashMassage],
  props: {
    anketaId: {
      required: false,
      default: 1,
      type: Number
    }
  },
  emits: ['closeEditModal'],
  setup() {
    const {cookies} = useCookies();
    return {cookies};
  },
  data() {
    return {
      idleTimer: null,
      minHoursSumm: 3000,
      user: this.$store.getters.getUser,
      apiUrl: this.$store.getters.getApiUrl,
      apiDomine: this.$store.getters['getApiDomine'],
      profile: {
        user_id: '',
        category_id: 8,
        name: '',
        age: '',
        height: '',
        weight: '',
        breast: '',
        clothing_size: '',
        shoe_size: '',
        not_younger: '',
        not_older: '',
        hair_color: '',
        intim_haircut: '',
        appearance: '',
        nationality: '',
        body_type: '',
        metro: '',
        metro_id: '',
        district_id: '',
        price_hour_app: '',//1 час аппарты
        price_two_hour_app: '' , //2 часа аппарты
        price_night_app: '', //ночь аппарты
        price_hour_night_app: '', //1 час ночи аппарты
        price_night_exit: '', //ночь выход
        price_hour_night_exit: '', //1 час ночи выход
        price_hour_exit: '', //1 час выход
        price_two_hour_exit: '', //2 часа выход
        description: '',
        tel: '+7',
        tg: '',
        city_id: '',
        city: '',
        whatsapp: '',
        currency_id: '1',
        checkService: [],
        status_id: 7,
        photo: [],
        video: '',
        tags: [],
        verification_code: '',
        verification_video: ''
      },
      validationFields: {
        'clothing_size': {
          valid: true,
          type: ''
        },
        'name': {
          valid: true,
          type: ''
        },
        'price_hour_app': {
          valid: true,
          type: ''
        },
        'price_hour_exit': {
          valid: true,
          type: ''
        },
        'price_hour_night_app': {
          valid: true,
          type: ''
        },
        'price_hour_night_exit': {
          valid: true,
          type: ''
        },
        'shoe_size': {
          valid: true,
          type: ''
        },
        'weight': {
          valid: true,
          type: ''
        },
        'city_id': {
          valid: true,
          type: ''
        }
      },
      validationPrice:{
                'price_hour_app': {
                    valid: true,
                    type: ''
                },
                'price_hour_exit': {
                    valid: true,
                    type: ''
                },
                'price_two_hour_app': {
                    valid: true,
                    type: ''
                },
                'price_two_hour_exit': {
                    valid: true,
                    type: ''
                },

                'price_hour_night_app': {
                    valid: true,
                    type: ''
                },
                'price_hour_night_exit': {
                    valid: true,
                    type: ''
                },
                'price_night_app': {
                    valid: true,
                    type: ''
                },
                'price_night_exit': {
                    valid: true,
                    type: ''
                },
            },
      loadingTel: false,
      errorTel: '',
      intlInput: false,
      intWaInput: false,
      check: false,
      profileStatus: [],
      cities: [],
      categories: [],
      currency: [],
      currencySymbol: '₽',
      service: {
        sex: [],
        dop: [],
        bdsm: [],
        massage: [],
        strip: [],
        extreme: [],
      },
      selectHairColor: [
        {id: 1, name: 'Блондинка'},
        {id: 2, name: 'Брюнетка'},
        {id: 3, name: 'Шатенка'},
        {id: 4, name: 'Рыжая'},
        {id: 5, name: 'Русая'},
        {id: 6, name: 'Цветная'},
        {id: 7, name: 'Лысая'},
      ],
      selectIntimHaircut: [
        {id: 1, name: 'Полная депиляция'},
        {id: 2, name: 'Аккуратная стрижка'},
        {id: 3, name: 'Натуральная'},
      ],
      selectAppearance: [
        {id: 1, name: 'Мулатка'},
        {id: 2, name: 'Славянская'},
        {id: 3, name: 'Татарка'},
        {id: 4, name: 'Китаянка'},
        {id: 5, name: 'Киргизка'},
        {id: 6, name: 'Казашка'},
        {id: 7, name: 'Метиска'},
        {id: 8, name: 'Азиатская'},
        {id: 9, name: 'Африканская'},
        {id: 10, name: 'Кавказская'},
        {id: 11, name: 'Латино'},
      ],
      selectStatus: '',
      selectNationality: [
        {id: 1, name: 'Белоруска'},
        {id: 2, name: 'Казашка'},
        {id: 3, name: 'Китаянка'},
        {id: 4, name: 'Кореянка'},
        {id: 5, name: 'Киргизка'},
        {id: 6, name: 'Русская'},
        {id: 7, name: 'Татарка'},
        {id: 8, name: 'Украинка'},
        {id: 9, name: 'Негритянка'},
        {id: 10, name: 'Кабардинка'},
        {id: 11, name: 'Узбечка'},
        {id: 11, name: 'Армянка'},
      ],
      selectBodyType: [
        {id: 1, name: 'Худая'},
        {id: 2, name: 'Стройная'},
        {id: 3, name: 'Спортивная'},
        {id: 4, name: 'Полная'},
      ],
      checkService: [],
      metros: [],
      metrosByCity: [],
      districts: [],
      isInvalid: false,
      error: '',
      errorHintMetro: '',
      loadingBtn: false,
      hMetro: false,
      metroName: '',
      districtName: '',
      selectedMoney: {
        id: 1,
        name: 'RUB',
        symbol: 'o'
      },
      choicePayContent: 'out',
      modalSaveQuest: false,
      showUpVideo: false,
    }
  },
  computed: {
    computedCity() {
      let computedArray = this.cities.filter(
          (item) => item.id !== this.profile.city
      );
      this.profile.city_id = this.profile.city.id
      return computedArray;
    },
    computedCategory() {
      let computedArray = this.categories.filter(
          (item) => item.id !== this.profile.category_id
      );
      return computedArray;
    },
    computedMoney() {
      let computedArray = this.currency.filter(
          (item) => item.id !== this.profile.currency_id
      );
      return computedArray;
    },
  },
  created() {
    if (this.$route.params && this.$route.params.id) {
      this.getGirlAnketa(this.$route.params.id)
    }else if(this.anketaId !== 1) {
      this.getGirlAnketa(this.anketaId)
    }else{
      this.profile.user_id = +this.user.profile.user_id
      this.profile.author_id = +this.user.user_id
      this.profile.status_id = 1
    }
  },
  beforeMount() {
    this.getAllCategory()
    this.getAllStatus()
    this.getAllCurrency()
    this.getAllService()
    this.generateRandomString()
  },
  mounted() {
      window.addEventListener('beforeunload', this.handlePageReload)
      this.idleTimer = setTimeout(() => {
        clearTimeout(this.idleTimer);
        if (this.$route.name == 'questionnaire') {
          this.modalSaveQuest = true
        }
      }, 3 * 60 * 1000)
      document.addEventListener('mousemove', this.resetTimer);
      document.addEventListener('keydown', this.resetTimer);
  },
  beforeDestroy() {
    window.removeEventListener('beforeunload', this.handlePageReload)
    clearTimeout(this.idleTimer);
    document.removeEventListener('mousemove', this.resetTimer)
    document.removeEventListener('keydown', this.resetTimer);
  },
  methods: {
    checkEmojis(data){
          let emojiRegex = /[\u{1F600}-\u{1F64F}\u{1F300}-\u{1F5FF}\u{1F680}-\u{1F6FF}\u{1F700}-\u{1F77F}\u{1F780}-\u{1F7FF}\u{1F800}-\u{1F8FF}\u{1F900}-\u{1F9FF}\u{1FA00}-\u{1FA6F}\u{2600}-\u{26FF}\u{2700}-\u{27BF}\u{2B50}\u{2B55}]/gu
          if (data == 'name') {
            if (emojiRegex.test(this.profile.name)) {
                this.profile.name = this.profile.name.replace(emojiRegex, '')
            }
          }
          if (data == 'text') {
            if (emojiRegex.test(this.profile.description)) {
                this.profile.description = this.profile.description.replace(emojiRegex, '')
            }
          }
          if (data == 'sex') {
            for (const key in this.service.sex) {
              if (emojiRegex.test(this.service.sex[key].description)) {
                this.service.sex[key].description = this.service.sex[key].description.replace(emojiRegex, '')
            }
            }
          }
          if (data == 'massage') {
            for (const key in this.service.massage) {
              if (emojiRegex.test(this.service.massage[key].description)) {
                this.service.massage[key].description = this.service.massage[key].description.replace(emojiRegex, '')
            }
            }
          }
          if (data == 'strip') {
            for (const key in this.service.strip) {
              if (emojiRegex.test(this.service.strip[key].description)) {
                this.service.strip[key].description = this.service.strip[key].description.replace(emojiRegex, '')
            }
            }
          }
          if (data == 'extrim') {
            for (const key in this.service.extreme) {
              if (emojiRegex.test(this.service.extreme[key].description)) {
                this.service.extreme[key].description = this.service.extreme[key].description.replace(emojiRegex, '')
            }
            }
          }
          if (data == 'sado') {
            for (const key in this.service.bdsm) {
              if (emojiRegex.test(this.service.bdsm[key].description)) {
                this.service.bdsm[key].description = this.service.bdsm[key].description.replace(emojiRegex, '')
            }
            }
          }
          if (data == 'other') {
            for (const key in this.service.dop) {
              if (emojiRegex.test(this.service.dop[key].description)) {
                this.service.dop[key].description = this.service.dop[key].description.replace(emojiRegex, '')
            }
            }
          }

        },
    changeCategory() {
      if (this.profile.city == 'Петербург' && this.profile.price_hour_app < 15000 && this.profile.price_hour_exit < 15000) {
        this.profile.category_id = 8
      } else if (this.profile.price_hour_app > 15000) {
        this.profile.category_id = 7
      }
      if (this.profile.city == 'Москва' && this.profile.price_hour_app < 20000 && this.profile.price_hour_exit < 20000) {
        this.profile.category_id = 8
      } else if (this.profile.price_hour_app > 20000) {
        this.profile.category_id = 7
      }
      if (this.profile.city == 'Екатеринбург' && this.profile.price_hour_app < 10000 && this.profile.price_hour_exit < 10000) {
        this.profile.category_id = 8
      } else if (this.profile.price_hour_app > 10000) {
        this.profile.category_id = 7
      }
    },
    handlePageReload() {
      this.modalSaveQuest = false
    },
    async saveQuest() {
      try {
        if (this.$route.params.id) {
          return
        }
        const res = await axios.post(this.apiUrl + 'api-girl/set-draft-profile&auth=' + this.user.username + ':' + this.user.auth_key,
            {
              user_id: this.user.user_id,
              category_id: this.profile.category_id,
              name: this.profile.name,
              age: this.profile.age,
              height: this.profile.height,
              breast: this.profile.breast,
              hair_color: this.profile.hair_color,
              metro: this.profile.metro,
              price_hour_app: this.profile.price_hour_app,
              price_two_hour_app: this.profile.price_two_hour_app,
              price_night_app: this.profile.price_night_app,
              status_id: 7,
              price_hour_exit: this.profile.price_hour_exit,
              price_two_hour_exit: this.profile.price_two_hour_exit,
              price_night_exit: this.profile.price_night_exit,
              description: this.profile.description,
              tel: this.profile.tel,
              tg: this.profile.tg,
              city_id: this.profile.city_id == '' || 'undefined' || undefined || null || 'null' ? 1 : this.profile.city_id,
              whatsapp: this.profile.whatsapp,
              currency_id: this.profile.currency_id,
              photo: [...this.profile.photo.map(p => {
                return {name: p.name}
              })],
              checkService: this.profile.checkService
            }
        )

      } catch (error) {
        console.log('QuestionnaireSave: ', error);
      }
    },
    resetTimer() {
      clearTimeout(this.idleTimer);
      this.idleTimer = setTimeout(() => {
        this.modalSaveQuest = true
        if (this.$route.name == 'questionnaire') {
          this.saveQuest()
        }
        if (this.$route.params.id) {
          this.modalSaveQuest = false
        }
      }, 3 * 60 * 1000);
    },
    setTel(tel = '') {
      if (!tel) return tel

      return tel.toString().replace(/[^0-9]/g, "");
    },
    switching(choice) {
      this.choicePayContent = choice
    },
    checkInput(event) {
      const regex = /^[a-zA-Z0-9@_-]+$/;
      const value = event.target.value;

      if (!regex.test(value)) {
        event.target.value = value.slice(0, -1); // Удаляем последний символ, если он не соответствует шаблону
      }
    },
    checkNumbers(input, a, b) {
      const value = input.value;
      const isValid = value && value >= a && value <= b;

      if (!isValid) {
        input.classList.add('is-invalid');
        this.error = 'invalid value'
      } else {
        input.classList.remove('is-invalid');
        this.error = ''
      }
    },
    setMaxSymbol(e, max, model = null) {
            const value = e.target.value
            if (this.profile.price_hour_app < 3000) {
                this.profile.price_hour_app = 3000
            }

            if (this.profile.price_two_hour_app < 3000) {
                this.profile.price_two_hour_app = 3000
            }
            if (this.profile.price_night_app < 3000) {
                this.profile.price_night_app = 3000
            }
            if (this.profile.price_hour_exit < 3000) {
                this.profile.price_hour_exit = 3000
            }
            if (this.profile.price_two_hour_exit < 3000) {
                this.profile.price_two_hour_exit = 3000
            }
            if (this.profile.price_night_exit < 3000) {
                this.profile.price_night_exit = 3000
            }
            if (this.profile.price_hour_night_app < 3000) {
                this.profile.price_hour_night_app = 3000
            }
            if (this.profile.price_hour_night_exit < 3000) {
                this.profile.price_hour_night_exit = 3000
            }

            if (this.profile.tel.length == 5) {
                let fourthChar = this.profile.tel.charAt(4)
                if (fourthChar == '7' || fourthChar == '8') {
                    this.profile.tel = this.profile.tel.slice(0, 4) + this.profile.tel.slice(5)
                }
            }

            if (value && value.length > max) {
                if (model) {
                    e.target.value = value.slice(0, -1)

                    if (model === 'tel' || model === 'whatsapp') {

                        this.profile[model] = value.replace(/\D/g, '').replace(/\^/g, '').slice(0, -1)
                    } else {
                        this.profile[model] = +value.slice(0, max)
                    }
                } else {
                    e.target.value = value.slice(0, -1)
                }
            }
        },
    addWhatsApp() {
      if (this.check) {
        this.profile.whatsapp = this.profile.tel
      } else {
        this.profile.whatsapp = ''
      }
    },
    getAllCategory() {
      axios
          .get(this.apiUrl + 'api/get-categories&auth=' + this.user.username + ':' + this.user.auth_key)
          .then((response) => {

            this.categories = response.data;
          })
    },
    getAllStatus() {
      axios
          .get(this.apiUrl + 'api/get-profile-status&auth=' + this.user.username + ':' + this.user.auth_key)
          .then((response) => {
            this.profileStatus = response.data;
          })
    },
    getAllCity(param) {
            if (param.length >= 1) {
                axios
                .get(this.apiUrl + 'api-hint/city&auth=' + this.user.username + ':' + this.user.auth_key + `&str=${param}`)
                .then((response) => {
                    console.log(response);
                   if (response.data.status != false) {
                     this.cities = response.data;
                    if (this.profile.city_id) {
                        this.addLocationHandler(this.cities.find(c => c.id === this.profile.city_id))
                        if (this.profile.metro_id) {
                            this.metroName = this.cities.find(c => c.id === this.profile.city_id)?.metro?.find(m => m.id === this.profile.metro_id)?.name
                        }
                        if (this.profile.district_id && this.profile.metro_id) {
                            this.districtName = this.cities.find(c => c.id === this.profile.city_id)?.districtWithMetro?.find(d => d.id === this.profile.district_id)?.name
                        }
                    }
                   }

                })
            }

        },
    getMetroByCityId(id) {
      axios
          .get(this.apiUrl + 'api/get-metro-by-city-id&auth=' + this.user.username + ':' + this.user.auth_key + '&id=' + id)
          .then((response) => {
            this.metrosByCity = response.data;
            for (let m of this.metrosByCity) {
              if (m.id === this.profile.metro_id) {
                this.metroName = m.name
              }
            }
          })
    },
    getAllCurrency() {
      axios
          .get(this.apiUrl + 'api/get-currencies&auth=' + this.user.username + ':' + this.user.auth_key)
          .then((response) => {
            this.currency = response.data;
          })
    },
    getAllService() {
      axios
          .get(this.apiUrl + 'api/get-profile-services&auth=' + this.user.username + ':' + this.user.auth_key)
          .then((response) => {
            for (let d of response.data) {
              d.checked = false
              d.description = ''
              setTimeout(() => {
                for (let cs of this.profile.checkService) {

                  if (parseInt(cs.service_id) === parseInt(d.id)) {
                    d.checked = true
                    d.price = cs.price
                    if (cs.description == null) {
                      d.description = ''
                    } else {
                      d.description = cs.description
                    }
                  }
                }
                if (d.category_id == 1) {
                  this.service.sex.push(d)
                }
                if (d.category_id == 2) {
                  this.service.dop.push(d)
                }
                if (d.category_id == 3) {
                  this.service.bdsm.push(d)
                }
                if (d.category_id == 4) {
                  this.service.massage.push(d)
                }
                if (d.category_id == 5) {
                  this.service.strip.push(d)
                }
                if (d.category_id == 6) {
                  this.service.extreme.push(d)
                }
              }, 700);
            }

          })
    },
    hintMetro() {
      if (this.metroName.length > 1) {
        let param = this.user.username + ':' + this.user.auth_key
            + '&city_id=' + this.profile.city_id
            + '&str=' + this.metroName
        axios
            .get(this.apiUrl + 'api-hint/metro&auth=' + param)
            .then((response) => {
              if (!response.data.status) {
                this.errorHintMetro = response.data.error
              }
              this.metrosByCity = response.data;
              this.hMetro = true

            })
      }
    },
    selectHintMetro(m) {
      this.profile.metro_id = m.id
      this.metroName = m.name
      this.hMetro = false
    },
    validAge() {
      if (this.profile.age < 18) {
        this.isInvalid = true
      } else {
        this.isInvalid = false
      }
    },
    addPhotoHandler(photos) {
      console.log('Получили событие загрузки. Данные: ', photos)
      console.log('this.profile.photo', this.profile.photo)

      if (this.$route.params.id) {
        this.profile.photo = photos
      } else {
        this.profile.photo = photos.map(p => {
          const names = p.name.split('/')
          const ln = names.length - 1
          return {name: names[ln]}
        })
      }
    },
    addVideoHandler(data, type = "") {
      if (type) {
        this.profile.verification_video = data.name;
        this.profile.status_id = 8
        this.showUpVideo = false
      } else {
        this.profile.video = data.name;
      }

      console.log('this.profile', this.profile)
    },
    addTagsHandler(tags) {
      this.profile.tags = tags;
    },
    async addLocationHandler(data) {
            if (data?.country_id) {
                this.profile.city_id = data.id
                this.profile.city = data.name
                this.metros = data.metro
                this.districts = data.districtWithMetro
            }
            if (data?.city_id && data?.district_id) {
                this.profile.metro_id = data.id
                this.profile.district_id = data.district_id
            } else if (data?.city_id) {
                this.profile.district_id = data.id
            }
        },
    addProfile() {
      this.changeCategory()
      this.profile.checkService = [];
      for (let s of this.service.sex) {
        // console.log('this.service.sex', this.service.sex)
        if (s.checked) {
          // console.log('this.profile.checkService', this.profile.checkService)
          this.profile.checkService.push(s)
        }
      }
      for (let s of this.service.bdsm) {
        if (s.checked) {
          this.profile.checkService.push(s)
        }
      }
      for (let s of this.service.dop) {
        if (s.checked) {
          this.profile.checkService.push(s)
        }
      }
      for (let s of this.service.massage) {
        if (s.checked) {
          this.profile.checkService.push(s)
        }
      }
      for (let s of this.service.extreme) {
        if (s.checked) {
          this.profile.checkService.push(s)
        }
      }
      for (let s of this.service.strip) {
        if (s.checked) {
          this.profile.checkService.push(s)
        }
      }
      if (this.isInvalid) {
        this.error = 'Возраст не может быть менее 18 лет'
      }

      Object.keys(this.validationFields).forEach(key => {
        if (this.profile[key] === '') {
          this.validationFields[key].valid = false
        } else {
          this.validationFields[key].valid = true
        }
      })

      Object.keys(this.validationPrice).forEach(key => {
                if (this.profile[key] < 3000) {
                    this.validationPrice[key].valid = false
                } else {
                    this.validationPrice[key].valid = true
                }
        })

      this.profile.tel = +this.setTel(this.profile.tel) || ''
      this.profile.whatsapp = +this.setTel(this.profile.whatsapp) || ''

      if ((Object.values(this.validationFields).find(v => !v.valid) !== undefined) || (this.error === 'invalid value')) {
        const message = {
          title: 'Ошибка при отправке анкеты',
          massage: 'Заполнены не все поля',
          type: 'danger'
        }
        this.$store.commit('setFlashMessage', message)
        return
      }
      if ((Object.values(this.validationPrice).find(v => !v.valid) !== undefined) || (this.error === 'invalid value')) {
              const message = {
                  title: 'Ошибка при отправке анкеты',
                  massage: 'Укажите сумму тарифа от 3000 рублей',
                  type: 'danger'
              }
              this.$store.commit('setFlashMessage', message)
              return
          }


      if (this.user.role.item_name === 'admin') {
        this.updateProfile(this.anketaId !== 1 ? `api-girl/update-profile&id=${this.anketaId}` : 'api-girl/set-profile')
      } else {
        this.updateProfile(this.$route.params.id ? `api-girl/update-profile&id=${this.profile.id}` : 'api-girl/set-profile')
      }
    },
    updateProfile(method) {
      const self = this
      axios.post(this.apiUrl + method + '&auth=' + this.user.username + ':' + this.user.auth_key, this.profile)
          .then(function (data) {
            if (!data.data.status) {
              const message = {
                title: 'Ошибка при отправке анкеты',
                massage: data.data.error,
                type: 'danger'
              }
              self.$store.commit('setFlashMessage', message)
            } else {
              const message = {
                title: 'Добавление анкеты',
                massage: 'Анкета успешно сохранена',
                type: 'success'
              }

              self.$store.commit('setFlashMessage', message)
              if (self.user.role.item_name === 'girl') {
                self.$router.push('/girl/lk')
              } else {
                self.$emit('closeEditModal')
              }
            }
          }).catch(function (data) {
        console.log('set-profile error: ', data)
        const message = {
          title: 'Добавление анкеты',
          massage: 'Ошибка при сохранении анкеты',
          type: 'danger'
        }

        self.$store.commit('setFlashMessage', message)
      }).finally(function () {
        self.error = ''
      })
    },
    selectChange(id) {
      this.profile.hair_color = this.$refs.selectHairColor.selectName
      this.profile.intim_haircut = this.$refs.selectIntimHaircut.selectName
      this.profile.appearance = this.$refs.selectAppearance.selectName
      this.profile.nationality = this.$refs.selectNationality.selectName
      this.profile.body_type = this.$refs.selectBodyType.selectName
    },
    selectChangeStatus(){
      this.profile.status_id = this.$refs.selectStatus.select_id
    },
    selectMoneyChange(id){
      this.currencySymbol = this.currency[id - 1].symbol
      this.profile.currency_id = id
    },
    generateRandomString() {
      if (this.$route.params && this.$route.params.id) return

      const possibleCharacters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
      let randomString = ''

      for (let i = 0; i < 5; i++) {
        randomString += possibleCharacters.charAt(Math.floor(Math.random() * possibleCharacters.length))
      }

      this.profile.verification_code = randomString
    },
    async getGirlAnketa(id) {
      try {
        const response = await axios.get(
            this.apiUrl + "api-girl/get-profile-by-id&auth="
            + this.user.username + ':' + this.user.auth_key
            + '&id=' + id
            + '&photo_size[width]=700'
            + '&photo_size[height]=700'
        )

        this.profile = {...response.data}

        if (this.profile.tags.length) {
          this.profile.tags = [...this.profile.tags.map(t => t.tag_id)]
        }

        this.profile.photo = [...this.profile.photo.map(p => {
          return {name: p.original_filename, originalFilename: p.original_filename}
        })]

        if (this.profile.video && this.profile.video.id) {
          this.profile.video
        }

        if (this.profile.city && this.profile.city.name) {
        }

        if (this.profile.metro_id) {
          this.profile.metro = ''
        }

        this.currencySymbol = response.data.currency.symbol
      } catch (error) {
        console.error('Download anketa error: ', error)

        const message = {
          title: 'Загрузка анкеты',
          massage: 'Ошибка при загрузке анкеты',
          type: 'danger'
        }
        this.$store.commit('setFlashMessage', message)
      }
    },
  },
}
</script>
<template>
  <h3 v-if="user.role.item_name === 'girl'">Добавление анкеты</h3>
  <h1 class="text-dark" v-else>Редактирование анкеты</h1>

    <div v-if="this.user.role.item_name === 'admin'" class="admin-controll mt-3 mb-5">
      <div class="drop-list">
        <label class="label">
          Статус Анкеты
          <ui-select ref="selectStatus" :select="profileStatus" @changeSelectVal="selectChangeStatus"
          />
        </label>
      </div>
      <div class="search-users">
        <label class="label">
          <UserSelect :labelText="'Сменить владельца анкеты: '"/> <!--Глючил id пользователя  " @selected="(e) => {this.profile.user_id = e.id} -->
        </label>
      </div>
      <div class="drop-list">
        <label class="label">
          Валюта
          <UiSelect
            ref="selectMoney"
            :select="currency"
            @changeSelectVal="selectMoneyChange"
          />
        </label>
      </div>
    </div>
    <!-- Загрузка photo-->
    <div class="row justify-content-between photo-upload">
      <div class="col-12 col-md-12 col-lg-4 mb-4 mb-lg-0">
        <div class="row">
          <div class="col-12 media">
            <h5>Загрузка фото</h5>
            <p class="photo-text">Перенесите фото на первое место, чтобы сделать его главным. Фото до 25Mb
              формата png, jpeg, jpg, webp
              Лимит по количеству фотографий 10 шт.</p>
          </div>
        </div>
        <upload-title-photo @upload-success="addPhotoHandler" :value="profile.photo"/>
      </div>
      <div class="col-12 col-md-6 col-lg-4 col-xxl-3 mb-4 mb-lg-0">
        <div class="row">
          <div class="col-12 media">
            <h5>Загрузите видеовизитку</h5>
            <p class="photo-text">Файл до 100Mb формата flv, avi, mov, mp4, mpeg, webm, mkv, 3gp</p>
          </div>
        </div>
        <upload-video-card @upload-video-success="(data) => addVideoHandler(data)" :value="profile.video" :photoArr="profile.photo"/>
      </div>

      <div v-if="user.role.item_name === 'girl'" class="col-12 col-md-6 col-lg-4 col-xxl-3 media">
        <div class="row">
          <div class="col-12 media">
            <h5>Проверочное видео</h5>
            <p class="photo-text">Загрузите проверочное видео-проходку с девушкой, чью анкету размещаете.
              Минимальный хронометраж ролика - 10 секунд. Размер файла до 200Mb.
              На видео девушка должна быть в полный рост, в руках у неё - листок А4, на котором написано:
              <b>{{ profile.verification_code }}</b>
            </p>
          </div>
        </div>
        <upload-video-card @upload-video-success="addVideoHandler" type="verification"/>
        <div class="row mt-3">
          <div class="col-12">
            <a href="">Посмотреть пример проверочного видео</a>
            <p class="sm-text">Проверочное видео не публикуется в общий доступ, оно нужно исключительно для
              проверки анкеты на
              соответствие заявленным фото, видит его только модератор</p>
          </div>
        </div>
      </div>
      <div v-else class="col-12 col-md-6 col-lg-4 col-xxl-3 media">
        <div class="row">
          <div class="col-12 media">
            <h5>Проверочное видео</h5>
            <p class="photo-text">Проверочный код:
              <b>{{ profile.verification_code }}</b>
            </p>
          </div>
        </div>
        <div class="media-ver-video">
          <div class="video-set" v-if="this.profile.verification_video">
            <video class="modal__video-m" controls preload="metadata" width="278px">
              <source :src="this.apiDomine + '/web/uploads/girls/verificationVideo/' + this.profile.verification_video">
            </video>
            <div class="row mt-2 justify-content-end actions">
              <button class="button hover-btn" @click="this.showUpVideo = true, this.profile.verification_video = ''">Обновить</button>
            </div>
          </div>

          <div v-else-if="this.showUpVideo || !this.profile.verification_video">
            <upload-video-card @upload-video-success="addVideoHandler" type="verification"/>
          </div>
        </div>
      </div>
    </div>
    <!--Личные данные-->
    <hr>
    <div class="row pt-4">

      <!--            Общая информация-->
      <div class="col-12 col-md-6 pe-md-3">
        <h4>Общая информация</h4>
        <div class="row">
          <div class="col-12">
            <input class="input-block__input"
                            :class="{ 'form-control is-invalid': !validationFields.name.valid }" type="text"
                            v-model="this.profile.name" placeholder="Имя (видно на сайте)"
                            @input="checkEmojis('name')"
                            @focus="validationFields.name.valid = true">
          </div>
          <div class="col-12 mt-3">
                        <textarea class="w-100 h-100 p-2" rows="5" placeholder="О себе (до 130 символов)"
                        @input="checkEmojis('text')"  v-model="this.profile.description"></textarea>
          </div>
          <div class="col-12 mt-3">
            <div class="row">
              <div class="col-12 col-xxl-4 d-flex align-items-center mb-3 mb-xxl-0 pe-xxl-0">
                <span class="ps-2"><i class="bi bi-telephone-fill me-1"></i></span>
                <input type="tel" name="phone" id="telephone" placeholder="_ ___ ___-__-__"
                                    class="form-control input-block__input" v-model="this.profile.tel"
                                    v-mask="'+# (###) ###-##-##'"
                                    @input="setMaxSymbol($event, 18, 'tel')">
              </div>
              <div class="col-12 col-xxl-4 d-flex mb-3 mb-xxl-0 pe-xxl-0">
                <img class="img-icon" src="@/assets/img/wh.png" alt="whatsApp">
                <input type="tel" name="whatsapp" id="whatsApp" placeholder="+ ___ ___-__-__"
                                    class="form-control input-block__input" v-model="this.profile.whatsapp"
                                    v-mask="'+# ### ###-##-##'" @input="setMaxSymbol($event, 16, 'whatsapp')">
              </div>
              <div class="col-12 col-xxl-4 d-flex">
                <img class="img-icon" src="@/assets/img/t.png" alt="TG">
                <input type="text" class="input-block__input ps-1" placeholder="@" @input="checkInput"
                       v-model="this.profile.tg">
              </div>
            </div>
          </div>
          <div class="col-12 mt-3">
            <div class="row ">
              <div class="col-12 col-xxl-4 mb-3 mb-xxl-0">
                <label class="label">
                  Город
                  <hint-input :array="cities" class="form-control"
                                        :class="{ 'is-invalid': !validationFields.city_id.valid }"
                                        :placeholder="'Начните вводить'" @select="addLocationHandler"
                                        @citynames="getAllCity"
                                        :current="profile.city?.name" @click="validationFields.city_id.valid = true" />
                </label>
              </div>
              <div class="col-12 col-xxl-4 d-flex mb-3 mb-xxl-0">
                <label class="label">
                  Метро
                  <hint-input :array="metros" class="form-control"
                              :placeholder="'Начните вводить'" @select="addLocationHandler"
                              :current="metroName"/>
                </label>
              </div>
              <div class="col-12 col-xxl-4 d-flex">
                <label class="label">
                  Район
                  <hint-input :array="districts" class="form-control"
                              :placeholder="'Начните вводить'" @select="addLocationHandler"
                              :current="districtName"/>
                  <!-- <input type="text" class="form-control" placeholder="Начните вводить"> -->
                </label>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!--            Тариф-->
      <div class="col-12 col-md-6 pt-4 pt-md-0">
        <h4>Тариф</h4>
        <div class="row">
          <div class="col-12 col-lg-6 p-1">
            <div class="card card-day h-100">
              <div class="card-body">
                <div class="row">
                  <div class="col-6">
                    <img src="@/assets/img/day1.png" alt="" class="img-fluid">
                  </div>
                  <div class="col-6 title justify-content-end">
                    1 час
                  </div>
                </div>
                <div class="row mt-2 align-items-center">
                  <div class="col-4 col-xl-3 text">
                    У меня
                  </div>
                  <div class="col-8 col-xxl-6 price">
                    <input type="number" v-model.lazy="profile.price_hour_app"
                        placeholder="Минимум 3000 рублей"
                        @input="setMaxSymbol($event, 6, 'price_hour_app')">
                    <span class="">{{ currencySymbol }}</span>
                  </div>
                </div>
                <div class="row mt-2">
                  <div class="col-4 col-xl-3 text">
                    У тебя
                  </div>
                  <div class="col-8 col-xxl-6 price">
                    <input type="number" v-model.lazy="profile.price_hour_exit"
                                            @input="setMaxSymbol($event, 6, 'price_hour_exit')">
                    <span class="">{{ currencySymbol }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="col-12 col-lg-6 p-1">
            <div class="card card-day h-100">
              <div class="card-body">
                <div class="row">
                  <div class="col-6">
                    <img src="@/assets/img/day2.png" alt="">
                  </div>
                  <div class="col-6 title justify-content-end">
                    2 часа
                  </div>
                </div>
                <div class="row mt-2 align-items-center">
                  <div class="col-4 col-xl-3 text">
                    У меня
                  </div>
                  <div class="col-8 col-xxl-6 price">
                    <input type="number" v-model.lazy="profile.price_two_hour_app"
                                            @input="setMaxSymbol($event, 6, 'price_two_hour_app')">
                    <span class="">{{ currencySymbol }}</span>
                  </div>
                </div>
                <div class="row mt-2">
                  <div class="col-4 col-xl-3 text">
                    У тебя
                  </div>
                  <div class="col-8 col-xxl-6 price">
                    <input type="number" v-model.lazy="profile.price_two_hour_exit"
                                            @input="setMaxSymbol($event, 6, 'price_two_hour_exit')">
                    <span class="">{{ currencySymbol }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="row">
          <div class="col-12 col-lg-6 p-1">
            <div class="card card-night h-100">
              <div class="card-body">
                <div class="row ">
                  <div class="col-6">
                    <img src="@/assets/img/night1.png" alt="">
                  </div>
                  <div class="col-6 title justify-content-end">
                    1 час
                  </div>
                </div>
                <div class="row mt-2 align-items-center">
                  <div class="col-4 col-xl-3 text">
                    У меня
                  </div>
                  <div class="col-8 col-xxl-6 price">
                    <input type="number" v-model.lazy="profile.price_hour_night_app"

                      @input="setMaxSymbol($event, 6, 'price_hour_night_app')">
                    <span class="">{{ currencySymbol }}</span>
                  </div>
                </div>
                <div class="row mt-2">
                  <div class="col-4 col-xl-3 text">
                    У тебя
                  </div>
                  <div class="col-8 col-xxl-6 price">
                    <input type="number" v-model.lazy="profile.price_hour_night_exit"
                                            @input="setMaxSymbol($event, 6, 'price_hour_night_exit')">
                    <span class="">{{ currencySymbol }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="col-12 col-lg-6 p-1">
            <div class="card card-night h-100">
              <div class="card-body">
                <div class="row ">
                  <div class="col-6">
                    <img src="@/assets/img/night2.png" alt="">
                  </div>
                  <div class="col-6 title justify-content-end">
                    Ночь
                  </div>
                </div>
                <div class="row mt-2 align-items-center">
                  <div class="col-4 col-xl-3 text">
                    У меня
                  </div>
                  <div class="col-8 col-xxl-6 price">
                    <input type="number" v-model.lazy="profile.price_night_app"

@input="setMaxSymbol($event, 6, 'price_night_app')">
                    <span class="">{{ currencySymbol }}</span>
                  </div>
                </div>
                <div class="row mt-2">
                  <div class="col-4 col-xl-3 text">
                    У тебя
                  </div>
                  <div class="col-8 col-xxl-6 price">
                    <input type="number" v-model.lazy="profile.price_night_exit"
                                            :class="{ 'form-control is-invalid': !validationPrice.price_night_exit.valid }"
                                            @focus="validationPrice.price_night_exit.valid = true"
                                            @input="setMaxSymbol($event, 6, 'price_night_exit')">
                    <span>{{ currencySymbol }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!--            Параметры-->
    <div class="row mt-4">
      <div class="col-6 col-sm-3 col-lg-2 col-xxl-1 mb-3 mb-xxl-0">
        <label class="label">
          Возраст
          <input type="number" class="form-control" placeholder="" v-model="profile.age"
                 @blur="checkNumbers($event.target, 18, 90)">
        </label>
      </div>
      <div class="col-6 col-sm-3 col-lg-2 col-xxl-1 mb-3 mb-xxl-0">
        <label class="label">
          Грудь
          <input type="number" class="form-control" placeholder="" v-model="profile.breast"
                 @blur="checkNumbers($event.target, 0, 6)">
        </label>
      </div>
      <div class="col-6 col-sm-3 col-lg-2 col-xxl-1 mb-3 mb-xxl-0">
        <label class="label">
          Рост, см
          <input type="number" class="form-control" placeholder="" v-model="profile.height"
                 @blur="checkNumbers($event.target, 50, 250)">
        </label>
      </div>
      <div class="col-6 col-sm-3 col-lg-2 col-xxl-1 mb-3 mb-xxl-0">
        <label class="label">
          Вес, кг
          <input type="number" class="form-control" :class="{ 'is-invalid': !validationFields.weight.valid }"
                 placeholder="" v-model="profile.weight" @focus="validationFields.weight.valid = true"
                 @blur="checkNumbers($event.target, 30, 150)">
        </label>
      </div>
      <div class="col-12 col-sm-6 col-md-4 col-xxl-2 mb-3 mb-xxl-0">
        <label class="label">
          Размер одежды
          <input type="number" class="form-control"
                 :class="{ 'is-invalid': !validationFields.clothing_size.valid }" placeholder=""
                 v-model="profile.clothing_size" @focus="validationFields.clothing_size.valid = true"
                 @blur="checkNumbers($event.target, 1, 150)">
        </label>
      </div>
      <div class="col-12 col-sm-6 col-md-4 col-xxl-2 mb-3 mb-xxl-0">
        <label class="label">
          Размер обуви
          <input type="number" class="form-control"
                 :class="{ 'is-invalid': !validationFields.shoe_size.valid }" placeholder=""
                 v-model="profile.shoe_size" @focus="validationFields.shoe_size.valid = true"
                 @blur="checkNumbers($event.target, 10, 50)">
        </label>
      </div>
      <div class="col-12 col-sm-6 col-md-4 col-xxl-2 mb-3 mb-xxl-0">
        <label class="label">
          Интимная стрижка
          <ui-select ref="selectIntimHaircut" :select="this.selectIntimHaircut"
                     @changeSelectVal="selectChange" :value="profile.intim_haircut"/>
        </label>
      </div>
      <div class="col-12 col-sm-6 col-md-4 col-xxl-2 mb-3 mb-xxl-0">
        <label class="label">
          Цвет волос
          <ui-select ref="selectHairColor" :select="this.selectHairColor" @changeSelectVal="selectChange"
                     :value="profile.hair_color"/>
        </label>
      </div>
    </div>
    <div class="row mt-0 mt-xxl-4 pb-2 pb-md-5">
      <div class="col-12 col-sm-6 col-lg-4 col-xxl-2 mb-3 mb-xxl-0">
        <label class="label">
          Внешность
          <ui-select ref="selectAppearance" :select="this.selectAppearance" @changeSelectVal="selectChange"
                     :value="profile.appearance"/>
        </label>
      </div>
      <div class="col-12 col-sm-6 col-lg-4 col-xxl-2 mb-3 mb-xxl-0">
        <label class="label">
          Национальность
          <ui-select ref="selectNationality" :select="this.selectNationality" @changeSelectVal="selectChange"
                     :value="profile.nationality"/>
        </label>
      </div>
      <div class="col-12 col-sm-6 col-lg-4 col-xxl-2 mb-3 mb-xxl-0">
        <label class="label">
          Телосложение
          <ui-select ref="selectBodyType" :select="this.selectBodyType" @changeSelectVal="selectChange"
                     :value="profile.body_type"/>
        </label>
      </div>
      <div class="col-12 col-sm-6 col-lg-4 col-xxl-3 mb-3 mb-xxl-0">
        <div class="row">
          <div class="col-6 col-sm-5">
            <label class="label">
              Не моложе
              <input type="number" class="form-control" placeholder="" v-model="profile.not_younger"
                     @blur="checkNumbers($event.target, 18, 90)">
            </label>
          </div>
          <div class="col-6 col-sm-7 d-flex align-items-end">
            <span class="sm-text">(ограничение по возрасту)</span>
          </div>
        </div>
      </div>
      <div class="col-12 col-sm-6 col-lg-4 col-xxl-3 mb-3 mb-xxl-0">
        <div class="row">
          <div class="col-6 col-sm-5">
            <label class="label">
              Не старше
              <input type="number" class="form-control" placeholder="" v-model="profile.not_older"
                     @blur="checkNumbers($event.target, 18, 90)">
            </label>
          </div>
          <div class="col-6 col-sm-7 d-flex align-items-end">
            <span class="sm-text">(ограничение по возрасту)</span>
          </div>
        </div>
      </div>
    </div>
    <hr class="pb-2 pb-md-5">
    <!--            Теги-->
    <tags-to-profile @select-tags="addTagsHandler" :items="profile.tags"/>
    <hr class="pb-1">
    <!--            Сервис-->
    <div class="">
      <div class="row">
        <h3>Предпочтения</h3>
        <div class="col-12">
          <ul class="nav nav-pills mb-3 mt-3 justify-content-start justify-content-lg-center" id="pills-tab"
              role="tablist">
            <li class="nav-item" role="presentation">
              <button class="nav-link active hover-btn" id="pills-sex-tab" data-bs-toggle="pill"
                      data-bs-target="#pills-sex" type="button" role="tab" aria-controls="pills-sex"
                      aria-selected="true">Секс
              </button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link hover-btn" id="pills-massage-tab" data-bs-toggle="pill"
                      data-bs-target="#pills-massage" type="button" role="tab" aria-controls="pills-massage"
                      aria-selected="false">Массаж
              </button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link hover-btn" id="pills-strip-tab" data-bs-toggle="pill"
                      data-bs-target="#pills-strip" type="button" role="tab" aria-controls="pills-strip"
                      aria-selected="false">Стриптиз
              </button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link hover-btn" id="pills-extreme-tab" data-bs-toggle="pill"
                      data-bs-target="#pills-extreme" type="button" role="tab" aria-controls="pills-extreme"
                      aria-selected="false">Экстрим
              </button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link hover-btn" id="pills-bdsm-tab" data-bs-toggle="pill"
                      data-bs-target="#pills-bdsm" type="button" role="tab" aria-controls="pills-bdsm"
                      aria-selected="false">Садо-мазо
              </button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link hover-btn" id="pills-dop-tab" data-bs-toggle="pill"
                      data-bs-target="#pills-dop" type="button" role="tab" aria-controls="pills-dop"
                      aria-selected="false">Разное
              </button>
            </li>
          </ul>
          <div class="tab-content" id="pills-tabContent">
            <div class="tab-pane fade show active" id="pills-sex" role="tabpanel"
                 aria-labelledby="pills-sex-tab" tabindex="0">
              <div class="row">
                <div class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2" v-for="item in service.sex"
                     :key="item.id">
                  <div class="form-check d-flex">
                    <div class="mt-2 p-0 w-100 d-flex justify-content-between">
                      <div class="d-flex align-items-center">
                        <input class="form-check-input" type="checkbox" v-bind:value="item"
                               v-model="item.checked" :id="item.id">
                        <label class="form-check-label service-text ms-3"
                               for="flexCheckDefault">
                          {{ item.name }}
                        </label>
                      </div>
                    </div>
                  </div>
                  <textarea v-show="item.checked" class="w-90 mt-2 ms-4"
                            placeholder="Комментарий к услуге" v-model="item.description"
                            @input="setMaxSymbol($event, 150); checkEmojis('sex')"></textarea>
                </div>
              </div>
            </div>

            <div class="tab-pane fade" id="pills-massage" role="tabpanel"
                 aria-labelledby="pills-massage-tab" tabindex="0">
              <div class="row">
                <div class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2" v-for="item in service.massage"
                     :key="item.id">
                  <div class="form-check d-flex">
                    <div class="mt-2 p-0 w-100 d-flex justify-content-between">
                      <div class="d-flex align-items-center">
                        <input class="form-check-input" type="checkbox" v-bind:value="item"
                               v-model="item.checked" :id="item.id">
                        <label class="form-check-label service-text ms-3"
                               for="flexCheckDefault">
                          {{ item.name }}
                        </label>
                      </div>
                    </div>
                  </div>
                  <textarea v-show="item.checked" class="w-90 mt-2 ms-4"
                            placeholder="Комментарий к услуге" v-model="item.description"
                            @input="setMaxSymbol($event, 150); checkEmojis('massage')"></textarea>
                </div>
              </div>
            </div>

            <div class="tab-pane fade" id="pills-strip" role="tabpanel" aria-labelledby="pills-strip-tab"
                 tabindex="0">
              <div class="row">
                <div class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2" v-for="item in service.strip"
                     :key="item.id">
                  <div class="form-check d-flex">
                    <div class="mt-2 p-0 w-100 d-flex justify-content-between">
                      <div class="d-flex align-items-center">
                        <input class="form-check-input" type="checkbox" v-bind:value="item"
                               v-model="item.checked" :id="item.id">
                        <label class="form-check-label service-text ms-3"
                               for="flexCheckDefault">
                          {{ item.name }}
                        </label>
                      </div>
                    </div>
                  </div>
                  <textarea v-show="item.checked" class="w-90 mt-2 ms-4"
                            placeholder="Комментарий к услуге" v-model="item.description"
                            @input="setMaxSymbol($event, 150); checkEmojis('strip')"></textarea>
                </div>
              </div>
            </div>

            <div class="tab-pane fade" id="pills-extreme" role="tabpanel"
                 aria-labelledby="pills-extreme-tab" tabindex="0">
              <div class="row">
                <div class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2" v-for="item in service.extreme"
                     :key="item.id">
                  <div class="form-check d-flex">
                    <div class="mt-2 p-0 w-100 d-flex justify-content-between">
                      <div class="d-flex align-items-center">
                        <input class="form-check-input" type="checkbox" v-bind:value="item"
                               v-model="item.checked" :id="item.id">
                        <label class="form-check-label service-text ms-3"
                               for="flexCheckDefault">
                          {{ item.name }}
                        </label>
                      </div>
                    </div>
                  </div>
                  <textarea v-show="item.checked" class="w-90 mt-2 ms-4"
                            placeholder="Комментарий к услуге" v-model="item.description"
                            @input="setMaxSymbol($event, 150); checkEmojis('extrim')"></textarea>
                </div>
              </div>
            </div>

            <div class="tab-pane fade" id="pills-bdsm" role="tabpanel" aria-labelledby="pills-bdsm-tab"
                 tabindex="0">
              <div class="row">
                <div class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2" v-for="item in service.bdsm"
                     :key="item.id">
                  <div class="form-check d-flex">
                    <div class="mt-2 p-0 w-100 d-flex justify-content-between">
                      <div class="d-flex align-items-center">
                        <input class="form-check-input" type="checkbox" v-bind:value="item"
                               v-model="item.checked" :id="item.id">
                        <label class="form-check-label service-text ms-3"
                               for="flexCheckDefault">
                          {{ item.name }}
                        </label>
                      </div>
                    </div>
                  </div>
                  <textarea v-show="item.checked" class="w-90 mt-2 ms-4"
                            placeholder="Комментарий к услуге" v-model="item.description"
                            @input="setMaxSymbol($event, 150); checkEmojis('sado')"></textarea>
                </div>
              </div>
            </div>

            <div class="tab-pane fade" id="pills-dop" role="tabpanel" aria-labelledby="pills-dop-tab"
                 tabindex="0">
              <div class="row">
                <div class="col-12 col-md-6 col-lg-4 col-xl-3 mt-2" v-for="item in service.dop"
                     :key="item.id">
                  <div class="form-check d-flex">
                    <div class="mt-2 p-0 w-100 d-flex justify-content-between">
                      <div class="d-flex align-items-center">
                        <input class="form-check-input" type="checkbox" v-bind:value="item"
                               v-model="item.checked" :id="item.id">
                        <label class="form-check-label service-text ms-3"
                               for="flexCheckDefault">
                          {{ item.name }}
                        </label>
                      </div>
                    </div>
                  </div>
                  <textarea v-show="item.checked" class="w-90 mt-2 ms-4"
                            placeholder="Комментарий к услуге" v-model="item.description"
                            @input="setMaxSymbol($event, 150); checkEmojis('other')"></textarea>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="row mt-5 justify-content-end actions">
      <button class="button hover-btn save"
              @click.stop="this.user.role.item_name === 'girl' ? $router.push('/girl/lk') : this.$emit('closeEditModal')">
        Закрыть
      </button>
      <button class="button hover-btn" @click="addProfile">Сохранить</button>
    </div>

  <div class="block" v-if="this.modalSaveQuest" @click="this.modalSaveQuest = false">
    <p class="block__text" @click.stop>
      Мы сохранили не заполненную анкету в черновиках. Можно позже вернуться к заполнению на странице <span
        @click="this.$router?.push('/girl/lk')">Мои анкеты</span>
    </p>
  </div>
</template>

<style scoped lang="scss">

.drop-list{
  max-width: 450px;
  width: 450px;
  flex: 1 0 calc(450px - 250px);
  max-height: 73px;
}

.search-users{
  max-width: 450px;
  width: 450px;
  flex: 1 0 calc(450px - 250px);
  max-height: 73px;
}

.admin-controll{
  gap: 20px 40px;
  display: flex;
}

@media (max-width: 925px) {
  .admin-controll{
    flex-direction: column;
  }
}@media (max-width: 540px) {
  .drop-list{
    max-width: 450px;
    width: 100%;
    flex: 0;
    max-height: 73px;
  }

  .search-users{
    max-width: 450px;
    width: 100%;
    flex: 0;
    max-height: 73px;
  }
}

.media-ver-video {
  max-width: 278px;
  width: 100%;
  border-radius: 5px;
}

.media-ver-video video {
  border-radius: 20px;
  height: 448px;
  object-fit: cover;
}

.block {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 9999;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;

  &__text {
    max-width: 350px;
    width: 100%;
    padding: 15px;
    border-radius: 10px;
    background: #fff;
    text-align: center;
    font-size: 20px;

    & span {
      text-decoration: underline;
      text-decoration-skip-ink: none;
      color: #4ba5ff;
      cursor: pointer
    }
  }
}

* {
  color: #525665;
}

h1 {
  color: #fff;
}

.media {
  color: #525665;
}

.media p {
  font-size: 14px;
  height: 160px;
}

@media screen and (min-width: 992px) and (max-width: 1100px) {
  .media h5 {
    font-size: 1rem;
  }

  .media p {
    font-size: 12px;
    min-height: 135px;
  }
}

@media screen and (min-width: 768px) and (max-width: 991px) {
  .media h5 {
    font-size: 1rem;
  }

  .media p {
    min-height: 180px;
  }

  .photo-upload > .col-12:nth-child(1) .media p {
    height: max-content;
    min-height: unset;
  }
}

@media screen and (max-width: 767px) {
  .media p {
    min-height: unset;
    height: max-content;
  }
}

.media a {
  text-decoration: none;
  cursor: pointer;
  font-size: 14px;
  color: #525665;
  font-weight: 600;
}

.img-icon {
  width: 35px;
  height: 35px;
}

.bi-telephone-fill {
  font-size: 21px;
  color: #525665;
}

h4 {
  color: #525665;
  margin-bottom: 1.5rem;
}

.label {
  font-size: 19px;
  color: #525665;
  width: 100%;
}

.card-day {
  background: #FBF8F7;
  border: none;
  border-radius: 12px;
}

.card-day .title {
  font-weight: 700;
  font-size: 24px;
  color: #201F36;
  display: flex;
  align-items: center;
}

.card-day .text {
  color: rgba(72, 76, 94, 0.6);
  font-size: 13px;
  padding-right: 0;
}

.card-day .price {
  font-weight: 700;
  font-size: 16px;
  color: #201F36;
  display: flex;
  align-items: center;
  gap: 10px;
}

.card-day input {
  height: 30px;
  background: #fff;
}

.card-night {
  background: #524B5F;
  border: none;
  border-radius: 12px;
}

.card-night .title {
  font-weight: 700;
  font-size: 24px;
  color: #fff;
  display: flex;
  align-items: center;
}

.card-night .text {
  color: #fff;
  font-size: 13px;
  padding-right: 0;
}

.card-night .price {
  font-weight: 700;
  font-size: 16px;
  color: #fff;
  display: flex;
  align-items: center;
  gap: 10px;
}

.card-night .price span {
  color: #fff;
}

.card-night input {
  height: 30px;
  background: #fff;
}


input {
  border: solid 1px #BDC1D1;
  border-radius: 12px;
  padding: 3px 15px;
  color: #333;
  line-height: 30px;
  width: 100%;
}


.invalid-input {
  border: 1px solid red !important;
}

input.check {
  width: 16px;
  max-width: 16px;
  height: 16px;
  border: solid 1px #BDC1D1;
  border-radius: 4px;
  cursor: pointer;
}

select {
  border: solid 1px #BDC1D1;
  border-radius: 10px;
  padding: 0 15px;
  color: #333;
  height: 30px;
  line-height: 30px;
  width: 100%;
  cursor: pointer;
}

textarea {
  border: solid 1px #BDC1D1;
  border-radius: 10px;
  padding: 0 15px;
  color: #333;
}

@media screen and (min-width: 768px) {
  textarea {
    max-width: calc(100% - 1.5rem);
  }
}

.sm-text {
  font-size: 12px;
}

.btn-dark {
  background-color: #39354B;
  color: #fff;
  border-radius: 14px;
}

.prof-title {
  font-size: 18px;
  color: #2D2F3A;
  font-weight: 600;
}

.form-check-input {
  width: 20px;
  height: 20px;
  padding: 0;
}

.form-check {
  margin-bottom: 0;
  min-height: 0;
}


.form-check-input:checked {
  background-color: #fff;
  border-color: #BDC1D1;
}

.form-check-input:checked[type=checkbox] {
  --bs-form-check-bg-image: url('data:image/svg+xml,%3csvg xmlns=%27http://www.w3.org/2000/svg%27 viewBox=%270 0 20 20%27%3e%3cpath fill=%27none%27 stroke=%27%231AD42C%27 stroke-linecap=%27round%27 stroke-linejoin=%27round%27 stroke-width=%273%27 d=%27m6 10 3 3 6-6%27/%3e%3c/svg%3e');
}

.form-check-input:focus {
  box-shadow: none;
}

.check-input {
  border: solid 1px #BDC1D1;
  border-radius: 50px;
  line-height: 26px;
  font-size: 14px;
  padding: 0 10px 0 10px;
  max-width: 67px;
}

.error-text {
  color: #980000;
  font-size: 16px;
}

.loading-tel {
  position: absolute;
  right: 51px;
  top: 12px;
  max-width: 55px;
}

.service-title {
  background-color: #8E7F7D;
  color: #FFF;
  font-size: 14px;
  line-height: 40px;
  border-radius: 9px;
  padding: 0 0 0 25px;
  display: flex;
  align-items: center;
}

.hint-metro {
  position: absolute;
  top: 30px;
  right: -52px;
  width: 180px;
  background: #fff;
  border: solid 1px #BDC1D1;
  border-radius: 10px;
  padding: 0 15px;
  color: #333;

}

.cursor {
  cursor: pointer;
}


.left-menu {
  border-right: 1px solid #535353;
}


.tab-pane img {
  max-width: 17px;
  margin-right: 15px;
}

.nav {
  position: relative;
  top: 0px;
}

.nav-pills .nav-link {
  background: #21232F;
  border-radius: 20px;
  border: none;
  color: #fff;
  margin-right: 15px;
  margin-left: 15px;

  @media (max-width: 991px) {
    margin-left: 0;
    margin-bottom: 10px;
  }
}


.nav-pills .nav-link.active {
  background: #21232F;
  border-radius: 20px;
  border: none;
  color: #1AD42C;
}

.name-service {
  font-size: 14px;
  font-weight: 400;
  color: #525665;
}

.name-service:hover {
  font-weight: 500;
}

.title-service {
  font-size: 24px;
  color: #525665;
}

.service-text {
  display: block;
  max-height: 20px;
}

.prev {
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  background: #fff;
  width: 34px;
  height: 34px;
  border: 1px solid #BDC1D1;
  border-radius: 5px;
}

.actions .button.save {
  background: #D9D9D9;
  color: #525665;
}

.actions .button {
  width: 130px;
  height: 30px;
  align-items: center;
  border-radius: 10px;
  background-color: #39354B;
  border: 1px solid #39354B;
  color: #fff;
  font-size: 12px;
  margin-right: 10px;
}
</style>