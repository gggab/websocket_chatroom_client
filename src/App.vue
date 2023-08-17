<template>
  <div id="app">
    <h1>WebSocket</h1>
    <to-do-item-vue label="WebSocket" :done="true"></to-do-item-vue>
  </div>
</template>

<script>
import ToDoItemVue from './components/ToDoItem.vue'

export default {
  name: 'App',
  components: {
    ToDoItemVue,
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
