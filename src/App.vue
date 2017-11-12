<template>
  <div id="app">
    <LiskPassphraseRecovery/>
  </div>
</template>

<script>
import LiskPassphraseRecovery from './components/LiskPassphraseRecovery';

export default {
  name: 'app',
  components: {
    LiskPassphraseRecovery,
  },
};
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
