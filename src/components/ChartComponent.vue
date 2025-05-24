<template>
    <div class="chart-container">
        <h2>График изменения курса валюты к рублю</h2>
        <div class="chartfield">
            <Line ref="myChart" id="my-chart-id2" :options="chartOptions" :data="chartData" />
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import { Line } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, Filler, PointElement, LineElement, CategoryScale, LinearScale } from 'chart.js'
ChartJS.register(Title, Tooltip, Legend, Filler, PointElement, LineElement, CategoryScale, LinearScale)
export default {
    name: 'TestChart',
    components: { Line },
    props: {
        dataForChartBuilding: {
            type: Object,
            required: true,
        },
    },
    emits: ['sendingData'],
    data() {
        return {
            fromParentObject: this.dataForChartBuilding,
            fullNameCurrency: '',
            componentShortName: '',
            chartData: {
                labels: [],
                datasets: []
            },
            chartOptions: {
                responsive: true,
                scales: {
                    y: {
                        //beginAtZero: true,
                        min: null,
                        max: null,
                        /*ticks: {
                            stepSize: 5,
                        }*/
                    }
                },
                maintainAspectRatio: false,
                plugins:{
                    
                }
            }
        }
    },
    beforeMount() {
        let today = new Date(Date.now());
        try {
            let i = 0;
            let pasteDate = 0;
            do {
                i = i + 1
                pasteDate = today.setDate(today.getDate() - i)
                this.chartData.labels.unshift(new Date(pasteDate).toLocaleString("ru-RU", { year: 'numeric', month: 'numeric', day: 'numeric' }));
                today.setDate(today.getDate() + i)
            } while (i < 30);
        } catch (error) {
            console.log("Ошибка: ")
        }
    },
    methods: {
        delay(ms) {
            return new Promise(resolve => setTimeout(resolve, ms));
        },
        async currencyForChart() {
            console.log(this.fromParentObject.masterCurrensyList)
            console.log("Запрос получен")
            let chartDataSetsArray = []
            this.chartData.datasets = []
            let i = 0;
            let firstChartColor = "71, 106, 212";
            let secondChartColor = "191, 43, 43";
            var colorArray = [firstChartColor, secondChartColor]
            let chartVerticalLineArray = [];
            let minimumValue;
            let maximumValue;
            let postDay = 0;
            let postMonth = 0;
            let postYear = 0;
            let dayMinus = 0;
            let monthMinus = 0;
            let yearMinus = 0;
            let historicalValues = [];
            let curenciesList = this.fromParentObject.masterCurrensyList;
            let shortCurrencyShortNameArray = this.fromParentObject.selectedCurrency;
            this.$emit('callAppFunction');
            for (let currencyName of shortCurrencyShortNameArray) {
                this.fullNameCurrency = curenciesList[currencyName].Name
                this.componentShortName = currencyName
                for (let splitedDate of this.chartData.labels) {
                    //console.log(splitedDate.split("."))
                    postDay = splitedDate.split(".")[0];
                    postMonth = splitedDate.split(".")[1];
                    postYear = splitedDate.split(".")[2];
                    try {
                        await axios.get(`https://www.cbr-xml-daily.ru//archive//${postYear}//${postMonth}//${postDay}//daily_json.js`)
                            .then(a => (historicalValues.push(a.data.Valute[this.componentShortName].Value)))
                        console.log(historicalValues)
                    } catch (error) {
                        if (historicalValues[0] == undefined) {
                            do {
                                console.log("Цикл по обработке ошибки")
                                //console.log(dayMinus, monthMinus, yearMinus)
                                if (postDay > 1 && postMonth > 1) {
                                    dayMinus++
                                } else if (postMonth > 1 && postDay <= 1) {
                                    monthMinus++
                                    postDay = 30;
                                    dayMinus = 0
                                } else if (postMonth == 1 && postDay == 1) {
                                    postMonth = 12;
                                    postDay = 30;
                                    yearMinus++
                                    monthMinus = 0;
                                }
                                try {
                                    await axios.get(`https://www.cbr-xml-daily.ru//archive//${postYear - yearMinus}//${postMonth - monthMinus}//${postDay - dayMinus}//daily_json.js`)
                                        .then(a => (historicalValues.push(a.data.Valute[this.componentShortName].Value)))
                                    error = false;
                                } catch (error) {
                                    console.log("данные не получены ДВА")
                                }
                                await this.delay(210);
                            } while (historicalValues[0] == undefined);
                        } else {
                            //console.log(historicalValues[0])
                            console.log("данные не получены")
                            historicalValues.push(historicalValues[historicalValues.length - 1]);
                        }
                    }
                    await this.delay(210);
                }
                minimumValue = Math.floor(Math.min(...historicalValues));
                maximumValue = Math.floor(Math.max(...historicalValues));
                chartVerticalLineArray.push(minimumValue, maximumValue)
                chartDataSetsArray.push(
                    {
                        data: [],
                        label: "",
                        tension: 0,
                        pointStyle: 'circle',
                        pointRadius: 1,
                        pointHoverRadius: 5,
                        borderWidth: 1,
                        label: `За ${curenciesList[this.componentShortName]['Nominal']} ${this.fullNameCurrency}`,
                        data: historicalValues,
                        fill: true,
                        borderColor: `rgb(${colorArray[i]})`,
                        /*backgroundColor: (context) => {
                            console.log("ФУНКЦИЯ ГРАДИЕНТА "+colorArray[i])
                            const ctx = context.chart.ctx;
                            var gradient = ctx.createLinearGradient(0, 0, 0, 300);
                            gradient.addColorStop(1, `rgba(${colorArray[i]}, 0.0)`);
                            gradient.addColorStop(0, `rgba(${colorArray[i]}, 1)`);
                            return gradient
                        },*/
                    }
                )
                historicalValues = [];
                i = i + 1
            };
            console.log('Рисуем график')
            console.log(this.chartData.datasets)
            console.log(this.chartData)
            minimumValue = Math.floor(Math.min(...chartVerticalLineArray) - 1);
            maximumValue = Math.floor(Math.max(...chartVerticalLineArray) + 1);
            this.chartOptions = {
                ...this.chartOptions,
                scales: {
                    y: {
                        min: minimumValue,
                        max: maximumValue,
                    }
                }
            };
            this.chartData = {
                ...this.chartData,
                datasets: chartDataSetsArray
            }
        },
    },
}




</script>

<style scoped></style>