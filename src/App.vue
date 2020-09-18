<template>
  <div id="app">
    <Fabric />
    <!-- <AlphaMask /> -->
  </div>
</template>

<script>
import Fabric from './components/Fabric'
// import AlphaMask from './components/AlphaMask'

export default {
  name: 'App',
  components: {
    Fabric,
    // AlphaMask
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  /* margin-top: 60px; */
}
</style>
