<template>
  <div id="app">
    <Roulette v-bind:count="10"
              v-bind:texts="texts"
              v-bind:colors="colors"
    />
  </div>
</template>

<script>
import Roulette from './components/Roulette.vue'

export default {
  name: 'app',
  components: {
    Roulette
  },
  data: function() {
    return {
      texts: [
        '100',
        '1',
        '10000',
        '-5',
        '5',
        'ALL IN',
        'BONUS!',
        'CONTINUE',
        '大吉',
        '末吉',
      ],
      colors: [
         '#75FF3B',
         '#E8BB35',
         '#FF6748',
         '#A635E8',
         '#2BA6FF',
         '#38FF83',
         '#E8E233',
         '#FF9E45',
         '#E8339F',
         '#292BFF',
      ],
    }
  }
}
</script>

<style lang="scss" scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
