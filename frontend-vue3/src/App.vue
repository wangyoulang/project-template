<!-- src/App.vue -->
<template>
  <div id="app">
    <BackendDemo />
  </div>
</template>

<script>
import BackendDemo from './components/BackendDemo.vue';

export default {
  name: 'App',
  components: {
    BackendDemo,
  },
};
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin-top: 60px;
}
</style>

