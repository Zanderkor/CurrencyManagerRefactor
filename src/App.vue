<template>

  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  </head>

  <body>
    <h1>Валютный менеджер</h1>
    <!-- <Test />-->
    <p id="pickStatus" ref="changePickStatus">Не выбрана валюта для обмена</p>
    <div class="body-box">
      <div class="functional-interface">
        <div class="input-and-pick">
          <div class="button-container">
            <div class="main-currencies-rates">
              <p>{{ majorCurrenciesRates }}</p>
            </div>
            <!--<button class="change" @click="valueAfterChange">Обмен на:</button>-->
          </div>
          <div class="input-container">
            <div class="inputShortName">
              <input class="currency" ref="currencyForTrade" placeholder="Валюта на руках" type="number"
                v-model="onHand" @input="valueAfterChange"></input>
              <select class="pick-currencyLeft" v-model="currencyTopSpace" @change="callBuildChart">
                <option disabled value="">Валюта которую хотите обменять</option>
                <option style="display: none">{{ currencyShortNameOnHand }}</option>
                <option>RUB - Российский рубль</option>
                <option v-for="curNameFirst in currencyArrayForSelect" :value="curNameFirst">{{ curNameFirst }}
                </option>
              </select>
              <!--<p class="shortname"> {{ currencyShortNameOnHand }} </p>-->
            </div>
            <div class="reverseburron">
              <button class="reverseChange" title="Наоборот" @click="reverseCurrency">
                <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="30" height="30" viewBox="0 0 64 64">
                  <path
                    d="M 32 6 C 17.641 6 6 17.641 6 32 C 6 33.147 6.0844688 34.273859 6.2304688 35.380859 L 10.357422 35.865234 C 10.131422 34.608234 10 33.321 10 32 C 10 19.869 19.869 10 32 10 C 38.615909 10 44.551673 12.942341 48.587891 17.580078 L 45.505859 21.652344 L 58 22 L 54.275391 10.068359 L 51.050781 14.328125 C 46.302784 9.2111633 39.530462 6 32 6 z M 53.642578 28.134766 C 53.868578 29.391766 54 30.679 54 32 C 54 44.131 44.131 54 32 54 C 25.383867 54 19.447695 51.057454 15.412109 46.419922 L 18.494141 42.347656 L 6 42 L 9.7246094 53.931641 L 12.945312 49.675781 C 17.692812 54.79188 24.469735 58 32 58 C 46.359 58 58 46.359 58 32 C 58 30.853 57.914531 29.726141 57.769531 28.619141 L 53.642578 28.134766 z">
                  </path>
                </svg>
              </button>
            </div>
            <div class="inputShortName">
              <input class="currency" ref="currencyTradedValue" placeholder="После обмена" v-model="exchangeValue">
              </input>
              <select class="pick-currencyRight" v-model="currencyBotSpace" @change="callBuildChart">
                <option disabled value="">Валюта на которую меняете</option>
                <option style="display: none"> {{ currencyShortName }} </option>
                <option>RUB - Российский рубль</option>
                <option v-for="curName in currencyArrayForSelect" :value="curName">{{ curName }}</option>
              </select>
              <!--<p class="shortname">{{ currencyShortName }}</p>-->
            </div>
          </div>
        </div>
        <ChartComponent :dataForChartBuilding="childObjectCurrency" @callAppFunction="valueAfterChange" ref="child" />
      </div>
      <div class="information-interface">

        <hr>
        <div class="wallet-news">
          <p>Новостной блок</p>
          <p style="color: red; font-size: 2rem;">!Раздел в разработке!</p>
        </div>
      </div>
    </div>
  </body>
</template>
<script>
import Test from './components/Test.vue'
import ChartComponent from './components/ChartComponent.vue';
import axios from 'axios'; //библиотека для работы с API

export default {
  components: { ChartComponent, Test },
  data() {
    return {
      result: null,
      majorCurrenciesRates: '',
      exchangeValue: null,
      onHand: null,
      curArr: [],
      currencyArrayForSelectOnHand: [],
      currencyArrayForSelect: [],
      currencyObjectsList: {},
      currencyTopSpace: '',
      currencyBotSpace: '',
      currencyShortNameOnHand: '',
      currencyShortName: '',      
      childObjectCurrency: {
        selectedCurrency: [],
        warnMessage: '',
        masterCurrensyList: ''
      },
    }
  },
  created() {
    this.axiosDataCurrencyArray();
  },
  methods: {
    async callBuildChart() {
      this.currencyShortNameOnHand = this.currencyTopSpace.substring(0, 3);
      this.currencyTopSpace = this.currencyShortNameOnHand;
      this.currencyShortName = this.currencyBotSpace.substring(0, 3);
      this.currencyBotSpace = this.currencyShortName;
      this.childObjectCurrency.selectedCurrency=[]; //при вызове функции массив всегда должен быть пустым
      console.log("Вызов колбэка")
      if (this.currencyBotSpace!='RUB' && this.currencyBotSpace!='' && this.currencyTopSpace!='RUB' && this.currencyTopSpace!='') {
      this.childObjectCurrency.selectedCurrency.push(this.currencyTopSpace, this.currencyBotSpace)
      } else if (this.currencyBotSpace!='RUB' && this.currencyBotSpace!='') {
        this.childObjectCurrency.selectedCurrency.push(this.currencyBotSpace)
      } else if (this.currencyTopSpace!='RUB' && this.currencyTopSpace!='') {
        this.childObjectCurrency.selectedCurrency.push(this.currencyTopSpace)
      }
      console.log(this.childObjectCurrency.selectedCurrency)
      this.childObjectCurrency.warnMessage = this.$refs.changePickStatus
      this.childObjectCurrency.masterCurrensyList = this.currencyObjectsList
      console.log(this.childObjectCurrency)
      await this.$refs.child.currencyForChart();
    },   
    async axiosDataCurrencyArray() {
      var currencyArrayShortNameList;
      await
        axios.get('https://www.cbr-xml-daily.ru/daily_json.js')
          .then(res => (this.currencyObjectsList = res.data.Valute));
      currencyArrayShortNameList = Object.keys(this.currencyObjectsList);
      currencyArrayShortNameList.forEach((element) => {
        this.currencyArrayForSelect.push(`${element} - ${this.currencyObjectsList[element]["Name"]}`)
      })
      try {
        let currentRateUsd
        let currentRateEur
        let currentRateCny
        let currentRateGbp
        const majorCurrenciesArray = ['USD', 'EUR', 'CNY', 'GBP']
        majorCurrenciesArray.forEach((shortName) => {
          let currentValue = this.currencyObjectsList[shortName]['Value'];
          let previousValue = this.currencyObjectsList[shortName]['Previous'];
          function trend(a, b) {
            if (a > b) { return `руб. ▲` }
            else if (a < b) { return `руб. ▼` }
            return ''
          }
          switch (shortName) {
            case 'USD': currentRateUsd = currentValue.toFixed(2) + trend(currentValue, previousValue);
            case 'EUR': currentRateEur = currentValue.toFixed(2) + trend(currentValue, previousValue);
            case 'CNY': currentRateCny = currentValue.toFixed(2) + trend(currentValue, previousValue);
            case 'GBP': currentRateGbp = currentValue.toFixed(2) + trend(currentValue, previousValue);
          }
        })
        this.majorCurrenciesRates = `Текущий курс основных валют:\nЕвро "€" - ${currentRateEur}\nЮань "¥" - ${currentRateCny}\nДоллар США "$" - ${currentRateUsd}\nФунт Стерлингов "£" - ${currentRateGbp}`
      } catch (error) {
        if (rateUsd == undefined || rateEur == undefined || rateCny == undefined || rateGbp == undefined) {
          this.majorCurrenciesRates = "Курс валют недоступен"
        }
      }
      this.currencyArrayForSelect.sort((a, b) => {
        const nameA = a.split(' - ')[1].toLowerCase();
        const nameB = b.split(' - ')[1].toLowerCase();
        if (nameA < nameB) {
          return -1;
        }
        if (nameA > nameB) {
          return 1;
        }
        return 0;
      });
      this.currencyArrayForSelectOnHand = this.currencyArrayForSelect;
      this.currencyShortName = this.currencyBotSpace.substring(0, 3);
    },

    valueAfterChange() {
      let currencyNominal = 0;
      let multiplyCof = 0;
      let onHandValue = 0;
      let curChangeValue = 0;
      const paragraphChange = this.$refs.changePickStatus;
      if (this.currencyShortName == '') {
        paragraphChange.style.visibility = 'visible'
        alert("Пожалуйста выбирете валюту для обмена")
        //console.log(this.currencyShortName)
      } else {
        paragraphChange.style.visibility = 'hidden'
        if (this.currencyShortNameOnHand == 'RUB' && this.onHand != '') {
          currencyNominal = this.currencyObjectsList[this.currencyShortName]['Nominal'];
          console.log(currencyNominal)
          multiplyCof = this.currencyObjectsList[this.currencyShortName]['Value'];
          this.exchangeValue = this.onHand / multiplyCof * currencyNominal;
          this.exchangeValue = this.exchangeValue.toFixed(2);
        } else if (this.currencyShortNameOnHand != 'RUB' && this.currencyShortName == 'RUB') {
          currencyNominal = this.currencyObjectsList[this.currencyShortNameOnHand]['Nominal'];
          multiplyCof = this.currencyObjectsList[this.currencyShortNameOnHand]['Value'];
          onHandValue = multiplyCof / currencyNominal;
          this.exchangeValue = this.onHand * onHandValue;
          this.exchangeValue = this.exchangeValue.toFixed(2);
        } else if (this.currencyShortNameOnHand != 'RUB' && this.currencyShortName != 'RUB') {
          if (this.currencyShortNameOnHand == '') { }
          else {
            onHandValue = this.currencyObjectsList[this.currencyShortNameOnHand]['Value'] / this.currencyObjectsList[this.currencyShortNameOnHand]['Nominal'];
            curChangeValue = this.currencyObjectsList[this.currencyShortName]['Value'] / this.currencyObjectsList[this.currencyShortName]['Nominal'];
            this.exchangeValue = this.onHand * onHandValue / curChangeValue;
            this.exchangeValue = this.exchangeValue.toFixed(2);
          }
        }
      }
    },
    reverseCurrency() {
      let reverseShortName = this.currencyShortName;
      this.currencyShortName = this.currencyShortNameOnHand;
      this.currencyShortNameOnHand = reverseShortName;
      this.onHand = this.exchangeValue;
      this.valueAfterChange()
    },

  },
}
</script>

<style scoped></style>
