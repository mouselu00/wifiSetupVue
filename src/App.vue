<template>
  <v-app id="app">
    <v-container>
      <v-layout align-center justify-center row fill-height>
        <v-card>
          <v-img width="50%" class="mx-auto" src="/static/PetDrinker-logo.png"/>
          <v-card-title>
            <v-card-text class="display-2">PetDrinker Setup</v-card-text>
            <SetupPage/>
          </v-card-title>
        </v-card>
      </v-layout>
    </v-container>
  </v-app>
</template>

<script>
import SetupPage from "./components/SetupPage";

export default {
  name: "App",
  components: {
    SetupPage
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
