<template lang="pug">
  div#app
    svg#background
      rect
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

  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  html {
    font-size: 100% !important;
  }

  #background {
    position: absolute;
    z-index: -9999;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
  }

  @font-face {
    font-family: 'arial-unicode';
    /*src: url('./assets/arial-unicode-ms.eot');*/
    /*src: url('./assets/arial-unicode-ms.eot?#iefix') format('embedded-opentype'),*/
    /*url('./assets/arial-unicode-ms.svg#my-font-family') format('svg'),*/
    /*url('./assets/arial-unicode-ms.woff') format('woff'),*/
    src: url('./assets/arial-unicode-ms.ttf') format('truetype');
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
