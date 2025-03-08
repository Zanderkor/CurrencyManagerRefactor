<template>
    <div class="chart-container">
        <h2>График изменения курса валюты к рублю</h2>
        <Line ref="myChart" id="my-chart-id2" :options="chartOptions" :data="chartData" />        
        <!--<div class="chartfield"></div>-->
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
        }
    },
    emits: ['sendingData'],
    data() {
        return {
            fromParentObject: this.dataForChartBuilding,
            fullNameCurrency: '',
            componentShortName: '',
            chartData: {
                labels: [], //значение оси Х
                datasets: [{
                    data: [], //значение оси Y          
                    label: "",
                    tension: 0,
                    pointStyle: 'circle',
                    pointRadius: 1,
                    pointHoverRadius: 5,
                    borderWidth: 1,
                    borderColor: 'rgb(169, 31, 31)',
                    backgroundColor: null,
                    fill: true,
                }]
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
            console.log("Запрос получен")
            this.componentShortName = this.fromParentObject.selectedCurrency.substring(0, 3);
            this.$emit('sendingData', this.componentShortName)            
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
            this.fullNameCurrency=this.fromParentObject.selectedCurrency.slice(5);      
            if (this.componentShortName != 'RUB') {
                if (this.componentShortName == '') {
                    //paragraphChange.style.visibility = 'visible'          
                } else {
                    //paragraphChange.style.visibility = 'hidden'
                    this.$emit('callAppFunction');
                    for (let splitedDate of this.chartData.labels) {
                        console.log(splitedDate.split("."))
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
                                    console.log(dayMinus, monthMinus, yearMinus)
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
                                    await this.delay(300);
                                } while (historicalValues[0] == undefined);
                            } else {
                                console.log(historicalValues[0])
                                console.log("данные не получены")
                                historicalValues.push(historicalValues[historicalValues.length - 1]);
                            }
                        }
                        await this.delay(300);
                    }
                    console.log('Рисуем график')
                    minimumValue = Math.floor(Math.min(...historicalValues)) - 1;
                    maximumValue = Math.floor(Math.max(...historicalValues)) + 1;
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
                        datasets: [{
                            ...this.chartData.datasets[0],
                            label: `За ${curenciesList[this.componentShortName]['Nominal']} ${this.fullNameCurrency}`,
                            data: historicalValues, // Обновляем только label
                            fill: true,
                            backgroundColor: (context) => {
                                const ctx = context.chart.ctx;
                                var gradient = ctx.createLinearGradient(0, 0, 0, 300);
                                gradient.addColorStop(1, 'rgba(169, 31, 31, 0.0)');
                                gradient.addColorStop(0, 'rgba(169, 31, 31, 1)');
                                return gradient
                            },
                        }]
                    };
                }
            }
        },
    },
}
</script>

<style></style>