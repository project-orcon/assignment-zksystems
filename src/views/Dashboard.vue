<template>
  <div class="dashboard">
    Dashboard
    <v-row>
      <v-col md="4" cols="12">
        <v-card height="100%">
          <v-card-title class="subtitle-2">TurnUp in MWh (current month)</v-card-title>
          <v-card-text class="teal--text display-2">507.54 MWh</v-card-text>
        </v-card>
      </v-col>
      <v-col md="4" cols="12" fill-height>
        <v-card height="100%">
          <v-card-title class="subtitle-2">TurnUp in Euro (current month)</v-card-title>
          <v-card-text class="teal--text display-2">&euro; 507.54</v-card-text>
        </v-card>
      </v-col>
      <v-col md="4" cols="12" fill-height>
        <v-card height="100%">
          <div class="d-flex flex-no-wrap justify-space-between">
            <div>
              <v-card-title class="subtitle-2">SGT5-4000F</v-card-title>
              
              <v-card-text>
                  <v-card-subtitle class="subtitle-2">10/12/18 15:15
              </v-card-subtitle><br>
              <span class="teal--text">
              Online 2 units remaining</span></v-card-text>
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
          <v-card-title class="subtitle-2"> TurnUp in MWh</v-card-title>
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
// @ is an alias to /src
</script>