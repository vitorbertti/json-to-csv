<template>
   <div id="app">
      <Converter msg="Welcome to JSON to CSV converter" />
   </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Converter from './components/Converter.vue';

@Component({
   components: {
      Converter,
   },
})
export default class App extends Vue {}
</script>

<style>
#app {
   margin: 0;
   padding: 0;
   box-sizing: border-box;
}
</style>
