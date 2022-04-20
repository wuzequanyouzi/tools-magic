<template>
  <div id="app">
    <HelloWorld
      url="http://127.0.0.1:5500/lib/test-umd.umd.js"
      name="123"
      @click="handleClick"
    />
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue';
export default {
  name: 'App',
  components: {
    HelloWorld,
  },
  methods: {
    handleClick() {},
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
