<template lang="pug">
  div#app
    router-view
</template>

<script>
  export default {
    name: 'app'
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
  body {
    background-color: #e3e3e3;
  }
  @font-face {
    font-family: 'arial-unicode';
    src: url('./assets/arial-unicode-ms.eot');
    src: url('./assets/arial-unicode-ms.eot?#iefix') format('embedded-opentype'),
    url('./assets/arial-unicode-ms.svg#my-font-family') format('svg'),
    url('./assets/arial-unicode-ms.woff') format('woff'),
    url('./assets/arial-unicode-ms.ttf') format('truetype');
    font-weight: normal;
    font-style: normal;
  }
  @font-face {
    font-family: 'arial-web';
    src: url('./assets/arial.eot');
    src: url('./assets/arial.eot?#iefix') format('embedded-opentype'),
    url('./assets/arial.svg#my-font-family') format('svg'),
    url('./assets/arial.woff') format('woff'),
    url('./assets/arial.ttf') format('truetype');
    font-weight: normal;
    font-style: normal;
  }
</style>
