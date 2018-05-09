<template>
  <div id="app">
    <calendar :show='true' @on-change='getDateVal'></calendar>
    <div>{{date}}</div>
  </div>
</template>

<script>
import Calendar from './components/Calendar'

export default {
  name: 'App',
  components: {
    Calendar
  },
  data () {
    return {
      date: ''
    }
  },
  methods: {
    getDateVal (val) {
      this.date = val
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
