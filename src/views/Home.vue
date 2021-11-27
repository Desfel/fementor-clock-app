<template>
  <div class="page-wrapper" :class="{'is-opened': isOpened, 'is-night' : stateOfDay === 'night'}">
    <div class="main-content-wrapper" v-bind:style="{ bottom: moreInfoTop + 'px' }">
      <section class="quote-section">
        <div class="quote-content">
          <p class="quote" v-text="`“${quotesArray[randomIndex].en}”`"></p>
          <p class="author" v-text="quotesArray[randomIndex].author"></p>
        </div>

        <a href="#" class="refresh-icon" @click="randomizeQuote">
          <svg width="18" height="18" xmlns="http://www.w3.org/2000/svg"><path d="M7.188 10.667a.208.208 0 01.147.355l-2.344 2.206a5.826 5.826 0 009.578-2.488l2.387.746A8.322 8.322 0 013.17 14.94l-2.149 2.022a.208.208 0 01-.355-.148v-6.148h6.52zm7.617-7.63L16.978.958a.208.208 0 01.355.146v6.23h-6.498a.208.208 0 01-.147-.356L13 4.765A5.825 5.825 0 003.43 7.26l-2.386-.746a8.32 8.32 0 0113.76-3.477z" fill="#FFF" fill-rule="nonzero" /></svg>
        </a>
      </section>

      <section class="clock-section">
        <div class="clock-wrapper">
          <p v-if="stateOfDay === 'morning'" class="greeting-text">
            <img src="@/assets/img/desktop/icon-sun.svg" alt="Daytime Sun">
            GOOD MORNING<span>, IT’S CURRENTLY</span>
          </p>
          <p v-else-if="stateOfDay === 'afternoon'" class="greeting-text">
            <img src="@/assets/img/desktop/icon-sun.svg" alt="Afternoon Sun">
            GOOD AFTERNOON<span>, IT’S CURRENTLY</span>
          </p>
          <p v-else-if="stateOfDay === 'night'" class="greeting-text">
            <img src="@/assets/img/desktop/icon-moon.svg" alt="Night Moon">
            GOOD EVENING<span>, IT’S CURRENTLY</span>
          </p>

          <div class="clock-time-wrapper">
            <p class="clock-time" v-text="formattedTime">11:37</p>
            <p class="clock-timezone" v-text="clockInfo.abbreviation">BST</p>
          </div>

          <p class="clock-location" v-text="`IN ${currentLocation.city}, ${currentLocation.country_code}`">IN LONDON, UK</p>
        </div>

        <a href="#" class="site-btn" :class="{'is-opened': isOpened}" @click="openInfoBlock">
          <p v-text="isOpened ? `Less` : `More`"></p>
          <span class="arrow">
          <img src="@/assets/img/desktop/icon-arrow-down.svg" />
        </span>
        </a>
      </section>
    </div>

    <section class="more-info-section" :class="{'is-opened': isOpened}">
      <div class="left-content info-block">
        <div class="info-wrapper">
          <p class="info-type">CURRENT TIMEZONE</p>
          <p class="info-value" v-text="clockInfo.timezone">Europe/London</p>
        </div>
        <div class="info-wrapper">
          <p class="info-type">Day of the year</p>
          <p class="info-value" v-text="clockInfo.day_of_year">295</p>
        </div>
      </div>

      <div class="right-content info-block">
        <div class="info-wrapper">
          <p class="info-type">Day of the week</p>
          <p class="info-value" v-text="clockInfo.day_of_week">5</p>
        </div>
        <div class="info-wrapper">
          <p class="info-type">Week number</p>
          <p class="info-value" v-text="clockInfo.week_number">6</p>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import axios from 'axios'
import moment from 'moment'

export default {
  data() {
    return {
      quotesArray: [{ en: '', author: '' }],
      clockInfo: [],
      formattedTime: '',
      stateOfDay: 'night',
      currentLocation: [],
      randomIndex: 0,
      isOpened: false,
      moreInfoTop: 0,
      errors: []
    }
  },
  head() {
    return {
      title: {
        inner: 'Home'
      },
      meta: [
        {
          name: 'description',
          content: `${this.appTitle} home page`,
          id: 'desc'
        }
      ]
    }
  },
  created() {
    axios.get('https://programming-quotes-api.herokuapp.com/Quotes?count=32')
      .then(quoteResponse => {
        this.quotesArray = quoteResponse.data
        this.randomIndex = Math.floor((Math.random()*31))
      })
      .catch(e => {
        this.errors.push(e)
      })

    axios.get('http://worldtimeapi.org/api/ip')
      .then(clockResponse => {
        this.clockInfo = clockResponse.data
        const urlSearchParams = new URLSearchParams(window.location.search)
        const urlParams = Object.fromEntries(urlSearchParams.entries())
        const urlParamsLength = Object.keys(urlParams).length
        let currentHour = moment(this.clockInfo.datetime).format('k')
        let currentMinute = moment(this.clockInfo.datetime).format('m')
        this.formattedTime = moment(this.clockInfo.datetime).format('kk:mm')

        if (urlParamsLength === 2) {
          currentHour = urlParams.hour
          currentMinute = urlParams.minute
          this.formattedTime = `${urlParams.hour}:${urlParams.minute}`
        }

        if (
          currentHour >= 5 &&
          currentMinute >= 0 &&
          (currentHour <= 11 && currentMinute <= 59)
        ) {
          this.stateOfDay = 'morning'
        } else if (
          currentHour >= 12 &&
          currentMinute >= 0 &&
          (currentHour <= 17 && currentMinute <= 59)
        ) {
          this.stateOfDay = 'afternoon'
        }
      })
      .catch(e => {
        this.errors.push(e)
      })

    axios.get(`https://api.freegeoip.app/json/?apikey=81722c80-4f42-11ec-9c76-5946d4511b39`)
      .then(geoResponse => {
        this.currentLocation = geoResponse.data
      })
      .catch(e => {
        this.errors.push(e)
      })
  },
  methods: {
    openInfoBlock(e) {
      e.preventDefault()
      this.isOpened = !this.isOpened
      this.moreInfoTop = this.isOpened ? document.querySelector('.more-info-section').clientHeight : 0
    },
    randomizeQuote(e) {
      e.preventDefault()
      this.randomIndex = Math.floor((Math.random()*31))
      console.log(this.randomIndex)
    }
  },
  computed: mapState('app', ['appTitle'])
}
</script>

<style lang="scss" scoped>
@import '@/theme/variables.scss';

.page-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  min-height: 100vh;
  background: url('../assets/img/desktop/bg-image-daytime.jpg') top left/cover no-repeat;
  overflow: hidden;

  @media (max-width: 1024px) {
    background: url('../assets/img/tablet/bg-image-daytime.jpg') center/cover no-repeat;
  }

  @media (max-width: 767px) {
    background: url('../assets/img/mobile/bg-image-daytime.jpg') center/cover no-repeat;
  }

  &.is-night {
    background: url('../assets/img/desktop/bg-image-nighttime.jpg') top left/cover no-repeat;

    @media (max-width: 1024px) {
      background: url('../assets/img/tablet/bg-image-nighttime.jpg') center/cover no-repeat;
    }

    @media (max-width: 767px) {
      background: url('../assets/img/mobile/bg-image-nighttime.jpg') center/cover no-repeat;
    }

    .more-info-section {
      background: rgba($black, .5);

      .info-block {

        &.left-content {
          border-right: 1px solid rgba($white, .25);
        }
        p {
          color: $white;
        }
      }
    }
  }

  &:before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    display: block;
    width: 100%;
    height: 100%;
    background: $black;
    opacity: .5;
    pointer-events: none;
  }

  &.is-opened {
    .quote-section {
      opacity: 0;
      pointer-events: none;
    }
  }

  p,
  a {
    font-weight: normal;
    font-size: 18px;
    line-height: 28px;
    font-family: $textFont;
    margin: 0;
    color: $white;
  }

  .site-btn {
    display: flex;
    align-items: center;
    padding: 8px 9px 8px 21px;
    background: $white;
    border-radius: 28px;
    letter-spacing: 5px;
    font-size: 16px;
    line-height: 28px;
    font-weight: bold;
    text-transform: uppercase;
    color: rgba($black, .5);

    p {
      color: rgba($black, .5);

      @media (max-width: 767px) {
        font-size: 12px;
        line-height: 14px;
        letter-spacing: 3.75px;
      }
    }

    @media (max-width: 767px) {
      padding: 3px 4px 3px 17px;
    }

    &:hover {
      .arrow {
        background: $greyHover;
      }
    }

    &.is-opened {
      .arrow {
        background: $greyHover;
        transform: rotate(180deg);
      }
    }

    .arrow {
      display: flex;
      align-items: center;
      justify-content: center;
      margin-left: 13px;
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: $grey;
      transition: all .3s ease-in-out;

      @media (max-width: 767px) {
        width: 32px;
        height: 32px;
      }

      img {
        @media (max-width: 767px) {
          width: 10px;
        }
      }
    }
  }

  .main-content-wrapper {
    position: relative;
    display: flex;
    flex-direction: column;
    width: 100%;
    min-height: 100vh;
    padding: 56px 10% 100px;
    transition: bottom .5s ease-in-out;

    @media (max-width: 1024px) {
      padding: 80px 64px 64px;
    }

    @media (max-width: 767px) {
      padding: 32px 26px 40px;
    }
  }

  .quote-section {
    position: relative;
    display: flex;
    align-items: flex-start;
    max-width: 540px;
    transition: opacity .3s ease-in-out;

    .quote-content {
      .quote {
        @media (max-width: 767px) {
          font-size: 12px;
          line-height: 22px;
        }
      }

      .author {
        font-weight: bold;
        margin-top: 14px;

        @media (max-width: 767px) {
          margin-top: 8px;
          font-size: 12px;
          line-height: 22px;
        }
      }
    }

    .refresh-icon {
      margin-top: 7px;
      margin-left: 16px;

      svg {
        opacity: .5;
        transition: opacity .3s ease-in-out;
      }

      &:hover {
        svg {
          opacity: 1;
        }
      }
    }
  }

  .clock-section {
    position: relative;
    margin-top: auto;
    display: flex;
    align-items: flex-end;

    @media (max-width: 1024px) {
      flex-direction: column;
      align-items: flex-start;
    }

    .clock-wrapper {
      .greeting-text {
        display: flex;
        align-items: center;
        letter-spacing: 4px;
        text-transform: uppercase;

        @media (max-width: 1024px) {
          letter-spacing: 3.6px;
        }

        @media (max-width: 767px) {
          font-size: 15px;
          line-height: 25px;
        }

        span {
          @media (max-width: 767px) {
            display: none;
          }
        }

        img {
          margin-right: 16px;
        }
      }

      .clock-time-wrapper {
        display: flex;
        align-items: flex-end;
        margin: 40px 0;

        .clock-time {
          font-weight: bold;
          font-size: 200px;
          line-height: 140px;
          letter-spacing: -5px;

          @media (max-width: 1024px) {
            font-size: 175px;
            line-height: 125px;
            letter-spacing: -4.375px;
          }

          @media (max-width: 767px) {
            font-size: 100px;
            line-height: 80px;
            letter-spacing: -2.5px;
          }
        }

        .clock-timezone {
          margin-left: 12px;
          font-weight: 300;
          font-size: 40px;
          line-height: 28px;

          @media (max-width: 767px) {
            font-size: 15px;
            line-height: 28px;
          }
        }
      }

      .clock-location {
        font-weight: bold;
        font-size: 24px;
        line-height: 28px;
        letter-spacing: 4.8px;
        text-transform: uppercase;

        @media (max-width: 1024px) {
          font-size: 18px;
          line-height: 28px;
          letter-spacing: 3.6px;
        }

        @media (max-width: 767px) {
          font-size: 15px;
          line-height: 28px;
          letter-spacing: 3px;
        }
      }
    }

    .site-btn {
      margin-left: auto;

      @media (max-width: 1024px) {
        margin-top: 80px;
        margin-left: 0;
      }

      @media (max-width: 767px) {
        margin-top: 48px;
      }
    }
  }

  .more-info-section {
    position: absolute;
    bottom: -100vh;
    margin-top: auto;
    display: flex;
    align-items: stretch;
    width: 100%;
    padding: 75px 10%;
    background: rgba($white, .5);
    transition: bottom .5s ease-in-out;

    &.is-opened {
      bottom: 0;
    }

    @media (max-width: 767px) {
      padding: 48px 26px;
      flex-direction: column;
    }

    .info-block {
      &.left-content {
        padding-right: 148px;
        border-right: 1px solid rgba($grey, .25);

        @media (max-width: 1024px) {
          padding: 0;
          margin-right: 80px;
          border: none !important;
        }

        @media (max-width: 767px) {
          margin: 0;
        }
      }

      &.right-content {
        margin-left: 94px;

        @media (max-width: 1024px) {
          margin: 0;
        }
      }

      .info-wrapper {
        @media (min-width: 768px) {
          &:last-child {
            margin-top: 42px;
          }
        }

        @media (max-width: 767px) {
          display: flex;
          align-items: center;
          margin-bottom: 16px;
        }
      }

      .info-type {
        font-weight: normal;
        font-size: 15px;
        line-height: 28px;
        letter-spacing: 3px;
        text-transform: uppercase;
        color: $grey;

        @media (max-width: 1024px) {
          font-size: 13px;
          line-height: 28px;
          letter-spacing: 2.6px;
        }

        @media (max-width: 767px) {
          font-size: 10px;
          line-height: 28px;
          letter-spacing: 2px;
        }

        @media (max-width: 320px) {
          font-size: 8px;
        }
      }

      .info-value {
        font-weight: bold;
        font-size: 56px;
        line-height: 68px;
        color: $grey;

        @media (max-width: 1024px) {
          font-size: 40px;
          line-height: 48px;
        }

        @media (max-width: 767px) {
          font-weight: bold;
          font-size: 20px;
          line-height: 24px;
          margin-left: auto;
        }

        @media (max-width: 320px) {
          font-size: 16px;
          line-height: 20px;
        }
      }
    }
  }
}
</style>
