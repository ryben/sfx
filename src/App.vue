<template>
  <div id="app">
    <Sfx msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import Sfx from './components/Sfx.vue'

export default {
  name: 'App',
  components: {
    Sfx
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
