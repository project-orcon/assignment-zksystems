<template>
  <div class="dashboard">
    Dashboard {{convertStringToNum("5,00")}}
    <v-row>
      <v-col md="4" cols="12">
        <v-card height="100%">
          <v-card-title class="subtitle-2">TurnUp in MWh (current month)</v-card-title>
          <v-card-text class="teal--text display-2">{{totalMWh()}} MWh</v-card-text>
        </v-card>
      </v-col>
      <v-col md="4" cols="12">
        <v-card height="100%">
          <v-card-title class="subtitle-2">TurnUp in Euro (current month)</v-card-title>
          <v-card-text class="teal--text display-2">&euro; {{totalEuro()}}</v-card-text>
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
          <v-card-text></v-card-text>
        </v-card>
      </v-col>
      <v-col md="6" cols="12">
        <v-card>
          <v-card-title class="subtitle-2">TurnUp in Euro</v-card-title>
          <v-card-text></v-card-text>
        </v-card>
      </v-col>
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
      mwhTotals: { minimum: 0, medium: 0, maximum: 0 },
      euroTotal: 0
    };
  },
  mounted: function() {
    this.loadCSV();
  },
  methods: {
    //takes a string of european number and converts to number
    convertStringToNum(numString) {
      var result = parseFloat(numString.replace(".", "").replace(",", "."));
      return result;
    },
    updateTotals() {
      //update the mwh and euro totals based on new data point.
      if (this.currentDataPoint.flame === FLAME_ON) {
        var irv = this.currentDataPoint.irv;
        if (MINIMUM_LOWER <= irv && irv <= MINIMUM_UPPER) {
          this.mwhTotals.minimum += this.currentDataPoint.mwh;
          console.log(
            "adding mimimum irv is " + this.currentDataPoint.irv + "mwh is ",
            this.currentDataPoint.mwh
          );
        } else if (MEDIUM_LOWER <= irv && irv <= MEDIUM_UPPER) {
          console.log(
            "adding medium irv is " + this.currentDataPoint.irv + "mwh is ",
            this.currentDataPoint.mwh
          );
          this.mwhTotals.medium += this.currentDataPoint.mwh;
        } else if (MAXIMUM_LOWER <= irv) {
          console.log(
            "adding maximum irv is " + this.currentDataPoint.irv + "mwh is ",
            this.currentDataPoint.mwh
          );
          this.mwhTotals.maximum += this.currentDataPoint.mwh;
        }
      }
    },
    totalMWh() {
      return (
        this.mwhTotals.minimum + this.mwhTotals.medium + this.mwhTotals.maximum
      );
    },
    totalEuro() {
      return (
        this.mwhTotals.minimum * PRICE_MIN_MINIMUM +
        this.mwhTotals.medium * PRICE_MIN_MEDIUM +
        this.mwhTotals.maximum * PRICE_MIN_MAXIMUM
      );
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
          vueInstance.currentDataPoint = {
            timestamp: data.timestamp,
            mwh: vueInstance.convertStringToNum(data.mw) / 60.0,
            irv: vueInstance.convertStringToNum(data.irv),
            flame: data.flame,
            eoh: data.eoh
          };
          vueInstance.updateTotals();
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