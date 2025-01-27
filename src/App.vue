<template>
  <div id="app">
    <MusicGrid />
  </div>
</template>

<script>
import MusicGrid from './components/MusicGrid.vue';

export default {
  name: 'App',
  components: {
    MusicGrid,
  },
};
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