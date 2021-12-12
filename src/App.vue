<template>
  <div id="app">
    <BaseHeader/>
    <BasePage/>
  </div>
</template>

<script>
import BaseHeader from './components/base-header.vue'
import BasePage from './pages/base-page.vue'

export default {
  name: 'App',
  components: {
    BasePage,
    BaseHeader
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
}
</style>
