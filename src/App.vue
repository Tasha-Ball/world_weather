<template>
  <div class="weather" id="app">
    <div class="container">

      <div class="weather__header">World Weather</div>

      <div class="weather__location">Watch weather in your current location</div>

      <button class="weather__add" >
        <img @click="searh=true"
          src="https://i.ibb.co/48s2L8B/1-Standard.png"
          alt="btn-add"
        />
      </button>

      <div class="popup-container"
        v-if="searh"
      >
        <div class="popup-body">
          <div class="search">
            <div class="search__header">Choose a city</div>
            <div class="search__info">To find city start typing and pick one from the suggestions</div>
            <input v-model.trim="city"
              :class="{invalid: ($v.city.$dirty && !$v.city.required) || !$v.city.alpha}"
              type="text"
              placeholder="Search city"
              class="search__input"
            >
            <small v-if="$v.city.$dirty && !$v.city.required"
              class="search__error">
              Поле не должно быть пустым. Введите название города (английская раскладка)
            </small>
            <small v-else-if="notFound"
              class="search__error">
              Город не найден. Введите корректное название города (английская раскладка)
            </small>
             <small v-else-if="dublicate"
              class="search__error">
              Данный город уже добавлен. Выберите другой город.
            </small>
            <div class="btn-menu">
              <div class="btn-menu__item">
                <button @click="clear"
                  class="btn-menu__secelct">
                  CLEAR
                </button>
              </div>
              <div class="btn-menu__item">
                <button class="btn-menu__secelct"
                  @click="searh=false; clear();">
                  CANCEL
                </button>
                <button class="btn-menu__secelct"
                  @click="validateSity">
                  ADD
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="all-city">
        <div v-for="(item, index) in weather"
          :key="index"
          :value="item"
        >
          <div v-if="item.name"
            class="main-city">
            <div class=" main-city__info main-city__info_bold ">
              {{item.name}}
            </div>
            <div v-if="item.sys.country =='RU'"
              class="main-city__info main-city__info_pb">
              Russia
            </div>
            <div class="main-city__container main-city__container_bb">
              <div class="main-city__info">
                Weather
              </div>
              <div class="main-city__info">
                {{item.weather[0].description}}
              </div>
            </div>
            <div class="main-city__container main-city__container_bb">
              <div class="main-city__info">
                temperature
              </div>
              <div class="main-city__info">
                {{item.temperature}} °C
              </div>
            </div>
              <div class="main-city__container main-city__container_bb">
                <div class="main-city__info">
                  Humidity
                </div>
                <div class="main-city__info">
                  {{item.main.humidity}} %
                </div>
              </div>
              <div class="main-city__container main-city__container_f">
                <div class="main-city__info main-city__info_p">
                  {{item.timelabel}}
                </div>
              </div>
              <div class="control-btn">
                <button @click="delItem(index)"
                  class="control-btn__remove">
                  REMOVE
                </button>
                <button  @click="getSity(item.name, index)"
                  class="control-btn__reload">
                  RELOAD
                </button>
              </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import { required, alpha } from 'vuelidate/lib/validators'

export default {
  name: 'App',

  data () {
    return {
      weather: [],
      searh: false,
      city: '',
      requestDone: false,
      notFound: false,
      dublicate: false

    }
  },

  validations: {
    city: { required, alpha }
  },

  methods: {

    validateSity () {
      if (this.$v.$invalid) {
        this.$v.$touch()
        return
      }
      this.getSity()
    },

    show () {
      this.add = true
    },

    getSity (name, index) {
      if (name) this.city = name
      localStorage.clear()
      fetch(`http://api.openweathermap.org/data/2.5/weather?q=${this.city},RU&appid=7a41ab91e4650db93a715fa7e390e998`)
        .then(res => res.json())
        .then(res => {
          if (res.cod === '404') {
            this.notFound = true
            return
          }
          let overlap = true
          this.weather.forEach((el) => { if (el.name.toLowerCase() === res.name.toLowerCase()) overlap = false })
          if (overlap || index >= 0) {
            if (index >= 0) {
              this.weather.splice(index, 1, { ...res, data: moment(), timelabel: 'now', temperature: Math.round(res.main.temp - 273.15) })
            } else {
              this.weather.push({ ...res, data: moment(), timelabel: 'now', temperature: Math.round(res.main.temp - 273.15) })
            }
            this.searh = false
            this.notFound = false
            localStorage.setItem('weather', JSON.stringify(this.weather))
            this.clear()
          } else {
            this.dublicate = true
          }
        })
        .catch(error => console.log(error))
    },

    delItem (index) {
      localStorage.clear()
      this.weather.splice(index, 1)
      localStorage.setItem('weather', JSON.stringify(this.weather))
    },

    settimelabel () {
      this.requestDone = false
      localStorage.clear()
      this.weather.forEach(element => {
        element.timelabel = moment(element.data).fromNow()
      })
      localStorage.setItem('weather', JSON.stringify(this.weather))
      this.requestDone = true
    },

    clear () {
      this.notFound = false
      this.dublicate = false
      this.city = ''
    }
  },

  watch: {
    requestDone: {
      handler: function () {
        setInterval(this.settimelabel, 5000)
      }
    }
  },

  created () {
    const weather = JSON.parse(localStorage.getItem('weather'))
    if (weather) {
      this.weather = weather
      this.requestDone = true
    }
  },

  beforeDestroy () {
    this.notFound = false
    clearInterval(this.settimelabel)
  }

}
</script>

<style lang="scss">

.container {
  width: 100%;
  max-width: 1565px;
  margin: 0 auto;
  position: relative;
  padding: 0;
}

.weather {
  height: 1080px;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 60px;
  font-family: 'Lato', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

    &__header {
      margin-top: 24px;
      margin-bottom: 32px;
      font-weight: 300;
      font-size: 70px;
      line-height: 80px;
      color: #1B1B1B;
    }

    &__location {
      font-size: 24px;
      line-height: 24px;
      color: #767676;
    }

    &__add {
      position: absolute;
      bottom: 1px;
      right: -105px;
      border: none;
      background: none;
      cursor: pointer;
    }
}

.all-city {
  height: auto;
  display: flex;
  flex-wrap: wrap;
  margin-top: 193px;
}

.main-city {
  width: 350px;
  height: 393px;
  display: flex;
  flex-direction: column;
  margin: 10px 20px;
  padding-left: 24px;
  padding-right: 24px;
  align-items: start;
  background: #FFFFFF;
  box-shadow: 0px 2px 10px rgba(10, 10, 10, 0.25);
  border-radius: 6px;
   box-sizing:border-box;

    &__container {
    display: flex;
    width: 100%;
    justify-content: space-between;

    &_f {
      justify-content: end;
    }

     &_bb {
        padding-top: 12px;
        padding-bottom: 12px;
        border-bottom: 1px solid #C4C4C4;;
      }
    }

    &__info {
      font-size: 18px;
      line-height: 24px;
      color: #1B1B1B;

      &_p {
        padding-top: 8px;
        padding-bottom: 32px;
        font-size: 16px;
        line-height: 24px;
        text-align: right;
        color: #A8A8A8;
      }

      &_bold {
        padding-top: 24px;
        padding-bottom: 15px;
        border: none;
        font-weight: bold;
        font-size: 32px;
        line-height: 38px;
        color: #1B1B1B;
      }

      &_pb {
      padding-bottom: 28px;
      }
    }

}

.control-btn {
  width: 100%;
  display: flex;
  justify-content: space-between;

   button {
    border: none;
    font-weight: bold;
    font-size: 16px;
    line-height: 24px;
    color: #9B51E0;
    background: none;
    cursor: pointer;
   }
}

.popup-container {
 position: fixed;
 width: 100%;
 height: 100%;
 background: rgba(11, 11, 11, 0.5);
 left: 0;
 top: 0;
}

.popup-body {
  min-height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px 10px;
}

.search {
  width: 651px;
  min-height: 359px;
  display: flex;
  flex-direction: column;
  align-items: start;
  padding: 24px;
  background: #FFFFFF;
  box-shadow: 0px 16px 24px rgba(0, 0, 0, 0.14), 0px 6px 30px rgba(0, 0, 0, 0.12), 0px 8px 10px rgba(0, 0, 0, 0.2);
  border-radius: 4px;

   &__header {
     padding-top: 24px;
     padding-bottom: 16px;
     font-weight: bold;
     font-size: 32px;
     line-height: 38px;
     color: #1B1B1B;
   }

   &__info {
     font-size: 24px;
     line-height: 24px;
     color: #767676;
     padding-bottom: 67px;
   }

   &__input {
     width: 100%;
     padding-top: 9px;
     padding-bottom: 9px;
     border: none;
     border-bottom: 1px solid #1B1B1B;
     font-size: 18px;
     line-height: 24px;
     color: #1B1B1B;
     outline:none;
   }

   &__input::placeholder {
     margin-top: 12px;
     font-size: 24px;
     line-height: 24px;
     color: #C1C1C1;
   }

   &__error {
     color: red
   }
}

.btn-menu {
  width: 100%;
  display: flex;
  justify-content: space-between;
  padding-top: 126px;

  &__item {
    width: 117px;
    display: flex;
    justify-content: space-between;
  }

  &__secelct {
    background: none;
    border: none;
    font-size: 16px;
    line-height: 24px;
    text-align: right;
    color: #C1C1C1;

    &:hover {
      cursor: pointer;
      font-weight: bold;
      color: blueviolet;
    }
  }
 }

@media(min-width:360px) and (max-width:766px) {

  .all-city{
    align-items: center;
    justify-content: center;
  }
  .main-city {
    margin: 20px 0;

  }
  .weather__add {
    padding-top: 60px;
    bottom: unset;
    right: 124px;

  }
}

@media(min-width:767px) and (max-width:1440px){

  .all-city{
    align-items: center;
    justify-content: center;
    margin-top: 53px;
  }

  .main-city {
    margin: 10px 13px;
  }

  .weather__add {
    display: contents;
    padding-top: 60px;
    bottom: unset;
    right: 338px;
  }
}

</style>
