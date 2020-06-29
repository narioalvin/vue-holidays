<template>
  <div id="app">
    <Overview />
  </div>
</template>

<script>
import Overview from './components/Overview.vue'

export default {
  name: 'App',
  components: {
    Overview
  }
}
</script>

<style>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}

@font-face {
  font-family: 'Poppins';
  src: url('./assets/font/Poppins-Regular.ttf');
  src: local('Poppins Regular'), local('Poppins-Regular'),
    url('./assets/font/Poppins-Regular.ttf') format('opentype');
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: 'Poppins';
  src: url('./assets/font/Poppins-Medium.ttf');
  src: local('Poppins Medium'), local('Poppins-Medium'),
    url('./assets/font/Poppins-Medium.ttf') format('opentype');
  font-weight: 600;
  font-style: normal;
}

@font-face {
  font-family: 'Poppins';
  src: url('./assets/font/Poppins-Bold.ttf');
  src: local('Poppins Bold'), local('Poppins-Bold'),
    url('./assets/font/Poppins-Bold.ttf') format('opentype');
  font-weight: bold;
  font-style: normal;
}

body {
  font-family: 'Poppins', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
