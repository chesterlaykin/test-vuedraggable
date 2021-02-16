<template>
  <div id="app">
     <draggable-test></draggable-test>
    
  </div>
</template>

<script>
import DraggableTest from './components/DraggableTest.vue'

export default {
  name: 'app',
  components: {
    DraggableTest
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
