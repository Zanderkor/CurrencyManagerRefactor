<template>

  <body>
    <h1>Валютный менеджер</h1>
    <div class="input-container">
      <div class="shortname">
        <p> {{ currencyShortNameOnHand }} </p>
      </div>
      <input class="currency" ref="currencyForTrade" placeholder="Валюта на руках" type="number" v-model="curInHand"
        @input="valueAfterChange()">
      </input>
      <div class="reverseburron">
        <button class="reverseChange" title="Наоборот" @click="reverseCurrency()"><svg
            xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="30" height="30" viewBox="0 0 64 64">
            <path
              d="M 32 6 C 17.641 6 6 17.641 6 32 C 6 33.147 6.0844688 34.273859 6.2304688 35.380859 L 10.357422 35.865234 C 10.131422 34.608234 10 33.321 10 32 C 10 19.869 19.869 10 32 10 C 38.615909 10 44.551673 12.942341 48.587891 17.580078 L 45.505859 21.652344 L 58 22 L 54.275391 10.068359 L 51.050781 14.328125 C 46.302784 9.2111633 39.530462 6 32 6 z M 53.642578 28.134766 C 53.868578 29.391766 54 30.679 54 32 C 54 44.131 44.131 54 32 54 C 25.383867 54 19.447695 51.057454 15.412109 46.419922 L 18.494141 42.347656 L 6 42 L 9.7246094 53.931641 L 12.945312 49.675781 C 17.692812 54.79188 24.469735 58 32 58 C 46.359 58 58 46.359 58 32 C 58 30.853 57.914531 29.726141 57.769531 28.619141 L 53.642578 28.134766 z">
            </path>
          </svg>
        </button>
      </div>
      <input class="currency" ref="currencyTradedValue" placeholder="После обмена" v-model="exchangeValue">
    </input>
      <div class="shortname">
        <p>{{ currencyShortName }}</p>
      </div>

    </div>
    <p id="pickStatus" ref="changePickStatus">Валюта не выбрана</p>
    <div class="button-container">
      <select class="pick-currency" v-model="currencyInfo" @change="currencySelectOnHand">
        <option disabled value="">Валюта которую хотите обменять</option>
        <option>RUB - Российский рубль</option>
        <option v-for="curNameFirst in fullCurrencyArrayOnHand" v-bind:value="curNameFirst">{{ curNameFirst }}</option>
      </select>
      <button class="change" @click="valueAfterChange()">Обмен</button>

      <select class="pick-currency" v-model="currencyInfoChart" @change="currencyForChart">
        <option disabled value="">Валюта на которую меняете</option>
        <option>RUB - Российский рубль</option>
        <option v-for="curName in fullCurrencyArray" v-bind:value="curName">{{ curName }}</option>
      </select>
    </div>
    <p class="main-currencies-rates">{{ runLine }}</p>
    <div class="chart-container">

      <h2>График иностранной валюты за последний месяц, относительно рубля</h2>
      <div class="chartfield">
        <Line ref="myChart" id="my-chart-id" :options="chartOptions" :data="chartData" />
      </div>

    </div>

  </body>



</template>

<script>

import axios from 'axios'; //библиотека для работы с API
//-->библиотека для работы с графиками и настройка графика
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  PointElement,
  LineElement,
  CategoryScale,
  LinearScale
}
  from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, PointElement, LineElement, CategoryScale, LinearScale)

export default {
  name: 'BarChart',
  components: { Line },
  data() {
    return {
      curNameChart: '',
      runLine: '',
      today: new Date(Date.now()),
      exchangeValue: null,
      curInHand: null,
      multiplyCof: null,
      currencyNominal: null,
      jaySon: ' ',
      curArr: [],
      fullCurrencyArrayOnHand: null,
      fullCurrencyArray: [],
      currencyList: '',
      currencyInfo: '',
      currencyInfoChart: '',
      currencyShortNamePick: '',
      currencyShortNameOnHand: '',
      currencyShortName: '',
      currencyFullName: '',
      currencyIndex: null,
      rateUsd: null,
      rateEur: null,
      rateGbp: null,
      rateCny: null,
      datacollection: null,
      dayDates: [],
      chartData: {
        labels: [],
        datasets: [{
          label: "",
          tension: 0,
          pointStyle: 'rectRounded',
          pointRadius: 5,
          pointHoverRadius: 7,
          borderColor: 'rgb(169, 31, 31)',
          data: [],
        }]
      },
      chartOptions: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true,
            ticks: {
              stepSize: 5,
            }
          }
        },
        responsive: true,
        maintainAspectRatio: false,
      }
    }
  },
  beforeMount() {
    try {
      let i = 0;
      let pasteDate = 0;
      do {
        i = i + 1
        pasteDate = this.today.setDate(this.today.getDate() - i)
        this.chartData.labels.unshift(new Date(pasteDate).toLocaleString("ru-RU", { year: 'numeric', month: 'numeric', day: 'numeric' }));
        this.today.setDate(this.today.getDate() + i)
      } while (i < 30);
    } catch (error) {
      console.log("Ошибка: ")
    }
  },

  created() {
    this.axiosDataCurrencyArray();
    this.worldCurrenciesRate();
  },
  methods: {
    currencySelectOnHand() {

      this.currencyShortNameOnHand = this.currencyInfo.substring(0, 3);

    },
    async currencyForChart() {
      const paragraphChange = this.$refs.changePickStatus;
      let dayMinus = 0;
      let monthMinus = 0;
      let yearMinus = 0;
      this.currencyShortName = this.currencyInfoChart.substring(0, 3);
      let historicalValues = [];
      console.log(this.currencyShortName);
      if (this.currencyShortName != 'RUB') {
        if (this.currencyShortName == '') {
          paragraphChange.style.visibility = 'visible'
          console.log(this.currencyShortName)
        } else {
          paragraphChange.style.visibility = 'hidden'
          this.valueAfterChange();
          for (let splitedDate of this.chartData.labels) {
            splitedDate.split(".")
            console.log(splitedDate.split(".")[1])
            try {
              await axios.get(`https://www.cbr-xml-daily.ru//archive//${splitedDate.split(".")[2]}//${splitedDate.split(".")[1]}//${splitedDate.split(".")[0]}//daily_json.js`)
                .then(a => (historicalValues.push(a.data.Valute[this.currencyShortName].Value)))
            } catch (error) {
              do {
                if (historicalValues[0] == undefined) {
                  if (splitedDate.split(".")[0] > 1 && splitedDate.split(".")[1] > 1) {
                    dayMinus++
                  } else if (splitedDate.split(".")[1] > 1 && splitedDate.split(".")[0] <= 1) {
                    monthMinus++
                    splitedDate.split(".")[0] = 30;
                  } else if (splitedDate.split(".")[1] == 1 && splitedDate.split(".")[0] == 1) {
                    splitedDate.split(".")[1] = 12;
                    splitedDate.split(".")[0] = 30;
                    yearMinus++
                  }
                  try {
                    await axios.get(`https://www.cbr-xml-daily.ru//archive//${splitedDate.split(".")[2] - yearMinus}//${splitedDate.split(".")[1] - monthMinus}//${splitedDate.split(".")[0] - dayMinus}//daily_json.js`)
                      .then(a => (historicalValues.push(a.data.Valute[this.currencyShortName].Value)))
                    error = false;
                  } catch (error) {
                    console.log("данные не получены ДВА")
                  }
                }
                setTimeout(500)
              } while (historicalValues[0] == undefined);
              console.log("данные не получены")
              historicalValues.push(historicalValues[historicalValues.length - 1]);
            }
            setTimeout(500);
            //this.chartData.datasets[0].data.push(historicalValues)
            //.then(a => (this.chartData.datasets[0].data.push(a.data.Valute[this.currencyInfoChart.substring(0,3)].value)))
          }
          this.chartData = {
            ...this.chartData,
            datasets: [{
              ...this.chartData.datasets[0],
              label: this.currencyInfoChart.slice(5),
              data: historicalValues // Обновляем только label
            }]
          };
        }
      }
    },
    async axiosDataCurrencyArray() {
      await
        axios.get('https://www.cbr-xml-daily.ru/daily_json.js')
          .then(res => (this.curArr = Object.keys(res.data.Valute)));
    },
    async axiosDataCurrencyList() {
      await
        axios.get('https://www.cbr-xml-daily.ru/daily_json.js')
          .then(pip => (this.currencyList = pip.data.Valute));
      this.curArr.forEach((element) => {
        this.fullCurrencyArray.push(`${element} - ${this.currencyList[element]["Name"]}`)
      })

      this.fullCurrencyArray.sort((a, b) => {
        const nameA = a.split(' - ')[1].toLowerCase();  // Получаем название после дефиса для a
        const nameB = b.split(' - ')[1].toLowerCase();  // Получаем название после дефиса для b
        if (nameA < nameB) {
          return -1;  // Если a идет перед b
        }
        if (nameA > nameB) {
          return 1;   // Если a идет после b
        }
        return 0;   // Если равны
      });
      this.fullCurrencyArrayOnHand = this.fullCurrencyArray;

    },
    async worldCurrenciesRate() {
      await this.axiosDataCurrencyList();
      try {
        this.rateUsd = this.currencyList["USD"]['Value'].toFixed(2)
        this.rateEur = this.currencyList["EUR"]['Value'].toFixed(2)
        this.rateCny = this.currencyList["CNY"]['Value'].toFixed(2)
        this.rateGbp = this.currencyList["GBP"]['Value'].toFixed(2)
        this.runLine = `Текущий курс основных валют: Доллар США "$" - ${this.rateUsd} р., Евро "€" - ${this.rateEur}р., Фунт Стерлингов "£" - ${this.rateGbp}р., Юань "¥" - ${this.rateCny}`
      } catch (error) {
        if (this.rateUsd == undefined || this.rateEur == undefined || this.rateCny == undefined || this.rateGbp == undefined) {
          this.runLine = "Курс валют недоступен"
          //console.log(error);
        }
      }
    },
    valueAfterChange() {
      let curInHandValue = 0;
      let curChangeValue = 0;
      const paragraphChange = this.$refs.changePickStatus;
      if (this.currencyShortName == '') {
        alert("Пожалуйста выбирете валюту для обмена")
        console.log(this.currencyShortName)
      } else {
        if (this.currencyShortNameOnHand == 'RUB' && this.curInHand != '') {
          this.currencyNominal = this.currencyList[this.currencyShortName]['Nominal'];
          this.multiplyCof = this.currencyList[this.currencyShortName]['Value'];
          this.exchangeValue = this.curInHand / this.multiplyCof * this.currencyNominal;
          this.exchangeValue = this.exchangeValue.toFixed(2);
        } else if (this.currencyShortNameOnHand != 'RUB' && this.currencyShortName == 'RUB') {
          this.currencyNominal = this.currencyList[this.currencyShortNameOnHand]['Nominal'];
          this.multiplyCof = this.currencyList[this.currencyShortNameOnHand]['Value'];
          curInHandValue = this.multiplyCof / this.currencyNominal;
          this.exchangeValue = this.curInHand * curInHandValue;
          this.exchangeValue = this.exchangeValue.toFixed(2);
        } else if (this.currencyShortNameOnHand != 'RUB' && this.currencyShortName != 'RUB') {
          curInHandValue = this.currencyList[this.currencyShortNameOnHand]['Value'] / this.currencyList[this.currencyShortNameOnHand]['Nominal'];
          curChangeValue = this.currencyList[this.currencyShortName]['Value'] / this.currencyList[this.currencyShortName]['Nominal'];
          this.exchangeValue = this.curInHand * curInHandValue / curChangeValue;
          this.exchangeValue = this.exchangeValue.toFixed(2);
        }

      }
    },
    reverseCurrency() {
      let reverseShortName = this.currencyShortName;
      this.currencyShortName = this.currencyShortNameOnHand;
      this.currencyShortNameOnHand = reverseShortName;
      //let reverseValue=this.exchangeValue;
      this.curInHand = this.exchangeValue;
      this.valueAfterChange()
    },
    /*curFullNameInfoFunc() {
      try {
        this.currencyFullName = this.currencyList[this.currencyShortName]["Name"]
        this.currencyInfoChart = `Вы хотите обменять ${this.currencyList[this.currencyShortName]["Name"]}`
      } catch (error) {
        this.currencyInfoChart = 'Валюта не выбрана'
      }
      console.log(this.currencyFullName)      
    },*/

  },
}



</script>

<style scoped></style>
