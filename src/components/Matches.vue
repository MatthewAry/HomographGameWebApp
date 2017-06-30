<template lang="pug">
  span(v-html="show")
</template>

<script>
  export default {
    name: 'match',
    props: {
      letterIndex: Number,
      word: String
    },
    computed: {
      show: function () {
        let pre = this.word.slice(0, this.letterIndex)
        let post = this.word.slice(this.letterIndex + 1)
        let char = this.word.charAt(this.letterIndex)
        return pre + '<span class="homograph">' + char + '</span>' + post
      }
    }
  }
</script>

<style lang="scss">
  .checkGame {
    > span {
      margin: 0 5px;
    }
    .homograph {
      color: darkred;
    }
  }
</style>
