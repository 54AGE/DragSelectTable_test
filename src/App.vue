<template>
  <div id="app">
    <HelloWorld :data="dataList"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import dataList from './data.js'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data() {
    return {
      dataList: [],
    }
  },
  mounted() {
    this.dataList = dataList;
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 100px;
  border: 1px solid black;
  background-color: grey;
}
</style>
