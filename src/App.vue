<template>
  <div id="app">
    <h1>Github User Search</h1>
    <p>A technical assignment from Ginetta for applicants. To be solved in under 3h.</p>
    <user-search />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import UserSearch from "./components/UserSearch.vue";

@Component({
  components: {
    UserSearch
  }
})
export default class App extends Vue {}
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  width: 500px;
  margin: 0 auto;
}
</style>
