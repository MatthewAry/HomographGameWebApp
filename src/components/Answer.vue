<template lang="pug">
  div.answers(v-html="show")
</template>

<script>
  import _ from 'lodash'
  export default {
    name: 'answer',
    props: {
      homograph: Array,
      homographData: Array
    },
    computed: {
      show: function () {
        let homographs = _.keyBy(this.homographData, function (o) {
          return String(o.word_index)
        })
        console.log(homographs)
        let answer = ''
        for (let i = 0; i < this.homograph.length; i++) {
          if (homographs[i]) {
            let word = homographs[i]
            answer += '<span class="this-is-a-homograph">'
            answer += this.homograph[i].slice(0, word.character_index)
            answer += '<strong>' + word.character_changed_to + '</strong>'
            answer += this.homograph[i].slice(word.character_index + 1)
            answer += '</span> '
          } else {
            answer += this.homograph[i] + ' '
          }
        }
        return answer
      }
    }
  }
</script>

<style lang="scss">
  .answers {
    margin-top: 10px;
    font-size: 20px;
    font-family: 'Roboto', sans-serif !important;
    strong {
      font-weight: 900 !important;
    }
    .this-is-a-homograph {
      color: #fff;
      background-color: #ff3f43;
      border-radius: 2px;
      padding: 1px 2px;
    }
  }
</style>
