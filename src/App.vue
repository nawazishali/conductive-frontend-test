<template>
  <div id="app">
    <Sankey/>
  </div>
</template>

<script>
import Sankey from './components/Sankey.vue'

export default {
  name: 'App',
  components: {
    Sankey
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
