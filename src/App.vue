<template>
  <div id="app">
    <Converter msg="Welcome to JSON / CSV converter" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Converter from "./components/Converter.vue";

@Component({
  components: {
    Converter
  }
})
export default class App extends Vue {}
</script>

<style>
#app {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
*,
button,
input {
  border: 0;
  outline: 0;
  font-family: "Roboto", sans-serif;
}
</style>
