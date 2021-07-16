<template>
  <div id="app">
    <Calendar />
  </div>
</template>

<script>
import Calendar from './views/Calendar.vue';

export default {
  name: "App",
  components: {
    Calendar,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 60px;
}
</style>
