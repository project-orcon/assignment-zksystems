<template>
  <div class="dashboard" style="max-width:1200px; margin:0 auto">
    <v-row v-show="loading" style="height:80vh">
      <v-col align-self="center" class="text-center teal--text">
        <v-progress-circular indeterminate color="teal" size="100">Loading...</v-progress-circular>
      </v-col>
    </v-row>
    <v-row v-show="!loading">
      <v-col md="4" cols="12">
        <v-card height="100%">
          <v-card-title class="subtitle-2">TurnUp in MWh (current month)</v-card-title>
          <v-card-text class="teal--text display-2">{{totalMWh.toFixed(2)}} MWh</v-card-text>
        </v-card>
      </v-col>
      <v-col md="4" cols="12">
        <v-card height="100%">
          <v-card-title class="subtitle-2">TurnUp in Euro (current month)</v-card-title>
          <v-card-text class="teal--text display-2">&euro; {{monthlyEuroTotal.toFixed(2)}}</v-card-text>
        </v-card>
      </v-col>
      <v-col md="4" cols="12">
        <v-card height="100%">
          <div class="d-flex flex-no-wrap justify-space-between">
            <div>
              <v-card-title class="subtitle-2">
                SGT5-4000F | GT10 &nbsp;
                <span class="grey--text">GT20</span>
              </v-card-title>
              <v-card-text>
                <v-card-subtitle class="subtitle-2">{{currentDataPoint.timestamp}}</v-card-subtitle>
                <br />
                <span class="teal--text">Online 2 units remaining</span>
              </v-card-text>
            </div>
            <v-avatar class="ma-2" size="125" tile>
              <v-img src="../assets/SGT5-4000F.gif"></v-img>
            </v-avatar>
          </div>
        </v-card>
      </v-col>
    </v-row>
    <v-row v-show="!loading">
      <v-col md="6" cols="12">
        <v-card>
          <v-card-title class="subtitle-2">TurnUp in MWh</v-card-title>
          <v-card-text>
            <canvas id="myChart" width="400" height="200" style="background-color:#eeeeee"></canvas>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col md="6" cols="12">
        <v-card>
          <v-card-title class="subtitle-2">TurnUp in Euro</v-card-title>
          <v-card-text>
            <canvas id="mylineChart" width="400" height="200" style="background-color:#eeeeee"></canvas>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
const FLAME_ON = "1,00";
const MINIMUM_LOWER = 100.5;
const MINIMUM_UPPER = 103.5;
const MEDIUM_LOWER = 103.5;
const MEDIUM_UPPER = 107.3;
const MAXIMUM_LOWER = 107.3;
const PRICE_MIN_MINIMUM = 0.42;
const PRICE_MIN_MEDIUM = 1.25;
const PRICE_MIN_MAXIMUM = 2.08;

export default {
  data() {
    return {
      loading: true,
      updateSignalPeriod: 0,
      currentDataPoint: { timestamp: "", mwh: 0, irv: 0, flame: "", eoh: "" },
      dailyMwhTotals: { minimum: 0, medium: 0, maximum: 0 },
      monthlyMwhTotals: { minimum: 0, medium: 0, maximum: 0 },
      currentHour: "",
      currentDay: "",
      currentDate: "",
      barChartIndex: -1,
      lineChartIndex: -1,
      currentMonth: "",
      barCanvas: {},
      barChart: {},
      lineCanvas: {},
      lineChart: {},
      stockMarketData: {},
      dailyMarketTotal: 0
    };
  },
  mounted: function() {
    this.loadChart();
    this.loadStockMarketCSV();
    this.loadCSV();
  },
  computed: {
    monthlyEuroTotal: function() {
      return (
        this.monthlyMwhTotals.minimum * PRICE_MIN_MINIMUM * 60 +
        this.monthlyMwhTotals.medium * PRICE_MIN_MEDIUM * 60 +
        this.monthlyMwhTotals.maximum * PRICE_MIN_MAXIMUM * 60
      );
    },
    dailyEuroTotal: function() {
      return (
        this.dailyMwhTotals.minimum * PRICE_MIN_MINIMUM * 60 +
        this.dailyMwhTotals.medium * PRICE_MIN_MEDIUM * 60 +
        this.dailyMwhTotals.maximum * PRICE_MIN_MAXIMUM * 60
      );
    },
    totalMWh: function() {
      return (
        this.monthlyMwhTotals.minimum +
        this.monthlyMwhTotals.medium +
        this.monthlyMwhTotals.maximum
      );
    }
  },
  methods: {
    //takes a string of european number and converts to number
    convertGermanStringToNum(numString) {
      var result = parseFloat(numString.replace(".", "").replace(",", "."));
      return result;
    },
    loadChart() {
      function newDate(days) {
        return moment()
          .add(days, "d")
          .toDate();
      }

      var barCanvas = document.getElementById("myChart");
      var lineCanvas = document.getElementById("mylineChart");
      this.lineCanvas = lineCanvas;
      this.barCanvas = barCanvas;
      var mybarChart = Chart.Bar(barCanvas, {
        data: {
          labels: [],
          datasets: [
            {
              label: "Minimum",
              data: [],
              backgroundColor: "#50BED7"
            },
            {
              label: "Medium",
              data: [],
              backgroundColor: "#3296B9" // yellow
            },
            {
              label: "Maximum",
              data: [],
              backgroundColor: "#095F87" // red
            }
          ]
        },
        options: {
          tooltips: {
            enabled: false
          },
          legend: { position: "bottom" },
          scales: {
            xAxes: [
              {
                //display: false hide labels
                categoryPercentage: 1.0, //no padding
                barPercentage: 0.9,
                offset: true,
                ticks: {
                  // display: false //this will remove only the label
                  autoSkip: true,
                  maxTicksLimit: 2,
                  maxRotation: 0 //don't rotate labesl
                },
                responsive: false,
                gridLines: {
                  offsetGridLines: true
                },

                stacked: true,
                beginAtZero: true
              }
            ],

            yAxes: [
              {
                stacked: true,
                ticks: {
                  beginAtZero: true
                }
              }
            ]
          }
        }
      });

      this.barChart = mybarChart;

      var mylineChart = new Chart(lineCanvas, {
        type: "line",
        data: {
          labels: [],
          datasets: [
            {
              label: "Customers Price",
              data: [],
              backgroundColor: "rgb(80,190,215,0.6)" //"#50BED7"
            },
            {
              label: "Market Price",
              data: [],
              backgroundColor: "#095F87" // red
            }
          ]
        },
        options: {
          layout: {
            padding: {
              left: 10
            }
          },
          tooltips: {
            enabled: false
          },
          elements: {
            point: {
              radius: 2
            }
          },
          legend: { position: "bottom" },
          scales: {
            xAxes: [
              {
                offset: true,
                ticks: {
                  maxRotation: 0
                },
                type: "time",
                time: {
                  parser: "DD.MM.YYYY",
                  displayFormats: { day: "MM/YY" },
                  unit: "month"
                }
              }
            ],

            yAxes: [
              {
                ticks: {
                  beginAtZero: true
                }
              }
            ]
          }
        }
      });
      this.lineChart = mylineChart;
    },
    initializeLineChart() {
      //clear all data from line chart
      this.lineChart.data.labels = [];
      this.lineChart.data.datasets[0].data = [];
      this.lineChart.data.datasets[1].data = [];
    },
    initializeBarChart(date) {
      //unlike the line chart - pushing labels with the current date doesn't work
      //for dynamic bar charts. Bars overlap each other. Bug in chartjs
      //as a result have to manually create bar graph and set labels equal to the
      //number of days in the month.

      var monthAsInteger = parseInt(date.split(".")[1]);
      var day = parseInt(date.split(".")[0]);

      //clear existing data from bar chart.
      this.barChart.data.labels = [];
      this.barChart.data.datasets[0].data = [];
      this.barChart.data.datasets[1].data = [];
      this.barChart.data.datasets[2].data = [];

      //set datasets index for bar graph

      //get month names to put min/max labels on x axis
      var monthName = moment(date, "DD.MM.YYYY").format("MMMM YYYY");
      var daysInMonth = moment(date, "DD.MM.YYYY").daysInMonth();
      var nextMonthName = moment(date, "DD.MM.YYYY")
        .add(1, "months")
        .format("MMMM YYYY");

      //add labels for all days of the month, and 0 values (label can have same name as
      //only the first and last labels will be shown)
      for (var x = 0; x < daysInMonth; x++) {
        this.barChart.data.labels.push(monthName);
        this.barChart.data.datasets[0].data.push(0);
        this.barChart.data.datasets[1].data.push(0);
        this.barChart.data.datasets[2].data.push(0);
      }
      this.barChart.data.labels.push(nextMonthName);
    },
    updateLineChartNewDay(date) {
      this.lineChart.data.labels.push(date);
      this.lineChart.data.datasets[0].data.push(0);
      this.lineChart.data.datasets[1].data.push(0);
      this.lineChart.update();
    },
    updateCharts() {
      this.barChart.data.datasets[0].data[
        this.barChartIndex
      ] = this.dailyMwhTotals.minimum;
      this.barChart.data.datasets[1].data[
        this.barChartIndex
      ] = this.dailyMwhTotals.medium;
      this.barChart.data.datasets[2].data[
        this.barChartIndex
      ] = this.dailyMwhTotals.maximum;
      this.barChart.update();

      this.lineChart.data.datasets[0].data[
        this.lineChartIndex
      ] = this.dailyEuroTotal;

      this.lineChart.data.datasets[1].data[
        this.lineChartIndex
      ] = this.dailyMarketTotal;
      this.lineChart.update();
    },
    updateDailyTotals(newMwh) {
      this.dailyMwhTotals.minimum += newMwh.minimum;
      this.dailyMwhTotals.medium += newMwh.medium;
      this.dailyMwhTotals.maximum += newMwh.maximum;
    },
    updateMonthlyTotals(newMwh) {
      this.monthlyMwhTotals.minimum += newMwh.minimum;
      this.monthlyMwhTotals.medium += newMwh.medium;
      this.monthlyMwhTotals.maximum += newMwh.maximum;
    },
    calculateNewData(currentDataPoint) {
      //takes current data point and returns mwh categories object.
      var result = { minimum: 0, medium: 0, maximum: 0 };
      if (currentDataPoint.flame === FLAME_ON) {
        var irv = currentDataPoint.irv;
        if (MINIMUM_LOWER <= irv && irv <= MINIMUM_UPPER) {
          result.minimum = currentDataPoint.mwh;
        } else if (MEDIUM_LOWER <= irv && irv <= MEDIUM_UPPER) {
          result.medium = currentDataPoint.mwh;
        } else if (MAXIMUM_LOWER <= irv) {
          result.maximum = currentDataPoint.mwh;
        }
      }

      return result;
    },

    updateDailyMarketTotal(date, hourString, currentMwhTotal) {
      var hour = parseInt(hourString) + 1; //hourString 0 to 23 map to 1 to 24
      if (date + "-" + hour in this.stockMarketData) {
        this.dailyMarketTotal +=
          this.stockMarketData[date + "-" + hour] * currentMwhTotal;
      }
    },

    loadStockMarketCSV() {
      var stockDict = {};
      var vueInstance = this;
      Papa.parse("/stockmarket_price_data.csv", {
        download: true,
        header: true,
        step: function(row) {
          row.data.hours = row.data.hours.replace("H", "");
          stockDict[row.data.day + "-" + row.data.hours] = parseFloat(
            row.data.price
          );
        },
        complete: function(results) {
          console.log("Finished:", results.data);
          vueInstance.stockMarketData = stockDict;
        }
      });
    },
    reformatTimestamp(original) {
      //formats timestamp from YYYY-MM-DD to DD.MM.YYYY
      var date = original.split(" ")[0];
      var time = original.split(" ")[1];
      var year = date.split("-")[0];
      var month = date.split("-")[1];
      var day = date.split("-")[2];

      return day + "." + month + "." + year + " " + time;
    },
    loadCSV() {
      //load the signal CSV file streaming one row at a time.
      var vueInstance = this;
      Papa.parse("/signal_data.csv", {
        download: true,
        header: true,
        step: function(row, parser) {
          parser.pause();
          vueInstance.loading = false;
          var data = row.data;
          var timestamp = vueInstance.reformatTimestamp(data.timestamp);

          //divide by 60 to convert mw minute into mwh, divide by 1000 to convert kW into MW
          vueInstance.currentDataPoint = {
            timestamp: timestamp,
            mwh: vueInstance.convertGermanStringToNum(data.mw) / 60000.0,
            irv: vueInstance.convertGermanStringToNum(data.irv),
            flame: data.flame,
            eoh: data.eoh
          };
          var newMwh = vueInstance.calculateNewData(
            vueInstance.currentDataPoint
          );

          var date = timestamp.split(" ")[0];
          var month = date.split(".")[1];
          var day = date.split(".")[0];
          var hour = timestamp.split(" ")[1].split(":")[0];

          if (month != vueInstance.currentMonth) {
            vueInstance.monthlyMwhTotals = {
              minimum: 0,
              medium: 0,
              maximum: 0
            };

            //reinitialise charts
            vueInstance.barChartIndex = parseInt(day) - 1 - 1;
            vueInstance.lineChartIndex = -1;

            vueInstance.initializeBarChart(date);
            vueInstance.initializeLineChart();
          }

          if (day != vueInstance.currentDay) {
            vueInstance.dailyMwhTotals = { minimum: 0, medium: 0, maximum: 0 };
            vueInstance.dailyMarketTotal = 0;

            //add new label to LineChart (Bar chart doesn't need one as labels created when initialized)
            vueInstance.updateLineChartNewDay(date);

            vueInstance.barChartIndex++;
            vueInstance.lineChartIndex++;
          }

          var totalMwhFromDataPoint =
            newMwh.minimum + newMwh.medium + newMwh.maximum;
          vueInstance.updateDailyMarketTotal(date, hour, totalMwhFromDataPoint);
          vueInstance.updateDailyTotals(newMwh);
          vueInstance.updateMonthlyTotals(newMwh);

          vueInstance.updateCharts();

          //update current values
          vueInstance.currentDate = date;
          vueInstance.currentMonth = month;
          vueInstance.currentDay = day;
          vueInstance.currentHour = hour;

          setTimeout(function() {
            parser.resume();
          }, vueInstance.updateSignalPeriod);
        },
        complete: function() {
          console.log("Finished streaming CSV file");
        }
      });
    }
  }
};
</script>