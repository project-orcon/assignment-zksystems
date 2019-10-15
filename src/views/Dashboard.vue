<template>
  <div class="dashboard">
    Dashboard {{convertStringToNum("5,00")}}
    <v-row>
      <v-col md="4" cols="12">
        <v-card height="100%">
          <v-card-title class="subtitle-2">TurnUp in MWh (current month)</v-card-title>
          <v-card-text class="teal--text display-2">{{totalMWh().toFixed(2)}} MWh</v-card-text>
        </v-card>
      </v-col>
      <v-col md="4" cols="12">
        <v-card height="100%">
          <v-card-title class="subtitle-2">TurnUp in Euro (current month)</v-card-title>
          <v-card-text class="teal--text display-2">&euro; {{totalEuro().toFixed(2)}}</v-card-text>
        </v-card>
      </v-col>
      <v-col md="4" cols="12">
        <v-card height="100%">
          <div class="d-flex flex-no-wrap justify-space-between">
            <div>
              <v-card-title class="subtitle-2">SGT5-4000F | GT10 GT20</v-card-title>
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
    <div class="row">
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
      {{hourlyMwhTotals}}
      <br />
      {{dailyMarketTotal}}
      <br />
      {{hourlyMwhTotal}}
    </div>
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
      updateSignalPeriod: 0,
      currentDataPoint: { timestamp: "", mwh: 0, irv: 0, flame: "", eoh: "" },
      hourlyMwhTotals: { minimum: 0, medium: 0, maximum: 0 },
      dailyMwhTotals: { minimum: 0, medium: 0, maximum: 0 },
      monthlyMwhTotals: { minimum: 0, medium: 0, maximum: 0 },
      currentHour: 0,
      currentDay: "",
      currentDayIndex: -1,
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
    this.loadCSV();
    this.loadStockMarketCSV();
  },
  computed: {
    currentMonth: function() {
      return currentDay.split("-")[1];
    },
    monthlyEuroTotal: function() {
      return (
        this.monthlyMwhTotals.minimum * PRICE_MIN_MINIMUM +
        this.monthlyMwhTotals.medium * PRICE_MIN_MEDIUM +
        this.monthlyMwhTotals.maximum * PRICE_MIN_MAXIMUM
      );
    },
    dailyEuroTotal: function() {
      return (
        this.dailyMwhTotals.minimum * PRICE_MIN_MINIMUM +
        this.dailyMwhTotals.medium * PRICE_MIN_MEDIUM +
        this.dailyMwhTotals.maximum * PRICE_MIN_MAXIMUM
      );
    },
    hourlyMwhTotal: function() {
      return (
        this.hourlyMwhTotals.minimum +
        this.hourlyMwhTotals.medium +
        this.hourlyMwhTotals.maximum
      );
    }
  },
  methods: {
    //takes a string of european number and converts to number
    convertStringToNum(numString) {
      var result = parseFloat(numString.replace(".", "").replace(",", "."));
      return result;
    },
    loadChart() {
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
                barPercentage: 0.06,
                stacked: true,
                offset: true,
                type: "time",
                time: {
                  parser: "DD.MM.YYYY",
                  displayFormats: { day: "MM/YY" },
                  unit: "month"
                }
              }
            ],

            yAxes: [{ stacked: true }]
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
              backgroundColor: "#50BED7"
            },
            {
              label: "Market Price",
              data: [],
              backgroundColor: "#095F87" // red
            }
          ]
        },
        options: {
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
                type: "time",
                time: {
                  parser: "DD.MM.YYYY",
                  displayFormats: { day: "MM/YY" },
                  unit: "month"
                }
              }
            ],

            yAxes: []
          }
        }
      });
      this.lineChart = mylineChart;
    },
    updateHourlyTotals(hour, newData) {
      //every hour have to update daily market price total.
      if (hour != this.currentHour) {
        this.updateDailyMarketTotal(this.currentDay, this.currentHour);
        this.currentHour = hour;
        this.hourlyMwhTotals = { minimum: 0, medium: 0, maximum: 0 };
      }
      this.hourlyMwhTotals.minimum += newData.minimum;
      this.hourlyMwhTotals.medium += newData.medium;
      this.hourlyMwhTotals.maximum += newData.maximum;
    },
    updateDailyTotals(day, newData) {
      if (day != this.currentDay) {
        console.log("new day %0 %0", day, this.dailyMwhTotals);
        this.dailyMwhTotals = { minimum: 0, medium: 0, maximum: 0 };
        this.dailyMarketTotal = 0;

        this.barChart.data.labels.push(day);
        this.barChart.data.datasets[0].data.push(0);
        this.barChart.data.datasets[1].data.push(0);
        this.barChart.data.datasets[2].data.push(0);
        this.barChart.update();

        this.lineChart.data.labels.push(day);
        this.lineChart.data.datasets[0].data.push(0);
        this.lineChart.data.datasets[1].data.push(0);
        this.lineChart.update();

        this.currentDay = day;
        this.currentDayIndex++;
      }
      this.dailyMwhTotals.minimum += newData.minimum;
      this.dailyMwhTotals.medium += newData.medium;
      this.dailyMwhTotals.maximum += newData.maximum;

      this.barChart.data.datasets[0].data[
        this.currentDayIndex
      ] = this.dailyMwhTotals.minimum;
      this.barChart.data.datasets[1].data[
        this.currentDayIndex
      ] = this.dailyMwhTotals.medium;
      this.barChart.data.datasets[2].data[
        this.currentDayIndex
      ] = this.dailyMwhTotals.maximum;
      this.barChart.update();

      this.lineChart.data.datasets[0].data[
        this.currentDayIndex
      ] = this.dailyEuroTotal;

      this.lineChart.data.datasets[1].data[
        this.currentDayIndex
      ] = this.dailyMarketTotal;
      this.lineChart.update();
    },
    updateTotals() {
      //update the mwh and euro totals based on new data point.
      var result = { minimum: 0, medium: 0, maximum: 0 };
      if (this.currentDataPoint.flame === FLAME_ON) {
        var irv = this.currentDataPoint.irv;
        if (MINIMUM_LOWER <= irv && irv <= MINIMUM_UPPER) {
          this.monthlyMwhTotals.minimum += this.currentDataPoint.mwh;
          result.minimum = this.currentDataPoint.mwh;
          console.log(
            "adding mimimum irv is " + this.currentDataPoint.irv + "mwh is ",
            this.currentDataPoint.mwh
          );
        } else if (MEDIUM_LOWER <= irv && irv <= MEDIUM_UPPER) {
          console.log(
            "adding medium irv is " + this.currentDataPoint.irv + "mwh is ",
            this.currentDataPoint.mwh
          );
          this.monthlyMwhTotals.medium += this.currentDataPoint.mwh;
          result.medium = this.currentDataPoint.mwh;
        } else if (MAXIMUM_LOWER <= irv) {
          console.log(
            "adding maximum irv is " + this.currentDataPoint.irv + "mwh is ",
            this.currentDataPoint.mwh
          );
          this.monthlyMwhTotals.maximum += this.currentDataPoint.mwh;
          result.maximum = this.currentDataPoint.mwh;
        }
      }

      return result;
    },
    totalMWh() {
      return (
        this.monthlyMwhTotals.minimum +
        this.monthlyMwhTotals.medium +
        this.monthlyMwhTotals.maximum
      );
    },

    totalEuro() {
      return (
        this.monthlyMwhTotals.minimum * PRICE_MIN_MINIMUM +
        this.monthlyMwhTotals.medium * PRICE_MIN_MEDIUM +
        this.monthlyMwhTotals.maximum * PRICE_MIN_MAXIMUM
      );
    },
    updateDailyMarketTotal(day, hour) {
      console.log("^^^", day + "-" + hour);
      console.log("^^^", this.stockMarketData[day + "-" + hour]);
      if (this.stockMarketData[day + "-" + hour]) {
        this.dailyMarketTotal +=
          this.stockMarketData[day + "-" + hour] * this.hourlyMwhTotal;
      }
    },

    loadStockMarketCSV() {
      var stockDict = {};
      var vueInstance = this;
      Papa.parse("/stockmarket_price_data.csv", {
        download: true,
        header: true,
        step: function(row) {
          console.log("Row:", row.data);
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
          var data = row.data;
          var timestamp = vueInstance.reformatTimestamp(data.timestamp);

          vueInstance.currentDataPoint = {
            timestamp: timestamp,
            mwh: vueInstance.convertStringToNum(data.mw) / 60.0,
            irv: vueInstance.convertStringToNum(data.irv),
            flame: data.flame,
            eoh: data.eoh
          };
          var diff = vueInstance.updateTotals();
          var date = timestamp.split(" ")[0];
          var hour = timestamp.split(" ")[1].split(":")[0];

          vueInstance.updateDailyTotals(date, diff);
          vueInstance.updateHourlyTotals(hour, diff);
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