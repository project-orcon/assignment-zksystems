<template>
  <v-app>
    <v-app-bar class="teal white--text" app>
      <div class="hidden-sm-and-down">
        <v-btn
          v-for="item in menu"
          :key="item.title"
          :to="item.link"
          text
          class="white--text"
          style="text-transform:none"
        >{{ item.title }}</v-btn>
      </div>
      <v-menu class="hidden-md-and-up teal white--text" offset-y>
        <template v-slot:activator="{ on }">
          <v-app-bar-nav-icon v-on="on" class="hidden-md-and-up white--text"></v-app-bar-nav-icon>
        </template>
        <v-list style="min-width:150px" class="teal white--text">
          <v-list-item v-for="item in menu" :key="item.icon">
            <v-list-item-content>
              <v-list-item-title class="white--text">{{ item.title }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-menu>
      <v-spacer></v-spacer>
      <v-toolbar-title>ZkSystems</v-toolbar-title>
    </v-app-bar>
   <v-content>
    <v-container fluid>
      <router-view></router-view>
    </v-container>
   </v-content>
      
  </v-app>
</template>

<script>
import HelloWorld from "./components/HelloWorld";

export default {
  name: "App",
  components: {
    HelloWorld
  },
  data: () => ({
    menu: [
      { icon: "home", title: "TurnUp Dashboard" },
      { icon: "info", title: "Invoice" },
      { icon: "info", title: "Smart Contract" },
      { icon: "info", title: "Transaction" },
      { icon: "info", title: "Machine Overview" }
    ]
  })
};
</script>
