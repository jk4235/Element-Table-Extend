<template>
  <div id="app">
    <img src="./assets/logo.png">
    <DefaultTable
      :get-data="tableData"/>
  </div>
</template>

<script>
import DefaultTable from './components/Table/Table'

export default {
  name: 'App',
  components: {
    DefaultTable
  },
  data () {
    return {
      tableData: []
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
