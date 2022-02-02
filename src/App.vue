<template>
  <div id="app">
    <SankeyDemo/>
  </div>
</template>

<script>
import SankeyDemo from '@/components/SankeyDemo.vue'

export default {
  name: 'App',
  components: {
    SankeyDemo
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
