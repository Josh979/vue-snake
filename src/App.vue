<template>
  <div class="font-bold uppercase my-2 text-4xl">Vue Snake</div>
  <Board />
  <Stats />
</template>

<script>
import Board from "./components/Board";
import Stats from "./components/Stats";
export default {
  name: 'App',
  components: {
    Board, Stats
  }
}
</script>

<style lang="scss">
html{
  background:black;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: limegreen;
  margin-top: 60px;
}
</style>
