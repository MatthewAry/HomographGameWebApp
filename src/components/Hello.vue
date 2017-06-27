<template lang="pug">
  div.hello
    h1 Spot The Homograph
    div.controls
      mu-raised-button(label="dialog", @click="openDialog")
      mu-raised-button(label="Get Homograph", @click="openHomograph")
    mu-dialog(:open="getHomographDialog", title="Game Settings", @close="close")
      | Difficulty {{difficulty}}
      mu-slider(name="Difficulty", :max="0.99", :min="0.85", :step="0.01", v-model="difficulty")
      | Frequency {{frequency}}
      mu-slider(name="Frequency", :min="0.1", :max="0.5", :step="0.01", v-model="frequency")
      mu-flat-button(slot="actions", secondary, @click="close", label="Cancel")
      mu-flat-button(slot="actions", primary, @click="getHomograph", label="Get Homograph")
    mu-dialog(:open="dialog", title="Spot The Homograph", @close="close")
      p
        | The general definition of a Homograph is seen when two or more words are
        | spelled the same but not necessarily pronounced the same or have different meanings.
      p
        | In our case, we're looking at words that look like they have the correct spelling,
        | but actually don't. These are words that have characters that look similar to the
        | the correct letter but actually is using a letter that a computer knows is different.
      p
        b
          | This is an experiment to see if you can spot the homograph. You will be presented
          | with a string of text that will contain homographs. To spot them all just tap on the
          | word you think looks a little off.
      mu-flat-button(slot="actions", primary, @click="close", label="OK")
    div.sequence(v-if="homograph.length")
      h2 Make your choices
      mu-flat-button.homograph(v-for="(word, index) in homograph", :label="word", @click="homographClick(index, $event)")
      h3 There are {{homographData.length}} words to find.

    div.selections(v-if="homograph.length")
      h2 Your selections
      mu-flat-button.homograph(v-for="index in selected", :label="homograph[index]")
    div.checkGame(v-if="selected.length")
      h2 Matches
      h3 {{homographData.length - selected.length}} left
      match(v-for="match in matches", :letter-index="match.index", :word="match.word")
</template>

<script>
  import _ from 'lodash'
  import MuRaisedButton from '../../node_modules/muse-ui/src/raisedButton/raisedButton'
  import MuFlatButton from '../../node_modules/muse-ui/src/flatButton/flatButton'
  import Match from './Matches.vue'
  export default {
    components: {
      Match,
      MuFlatButton,
      MuRaisedButton
    },
    name: 'hello',
    data () {
      return {
        difficulty: 0.85,
        frequency: 0.10,
        dialog: false,
        getHomographDialog: false,
        homograph: [],
        originalText: [],
        homographData: [],
        selected: [],
        matches: []
      }
    },
    methods: {
      openDialog () {
        this.dialog = true
      },
      close () {
        this.dialog = false
        this.getHomographDialog = false
      },
      openHomograph () {
        this.getHomographDialog = true
      },
      getHomograph () {
        let _this = this
        this.getHomographDialog = false
        this.$http.post('https://byui-homograph.appspot.com/getHomograph', {difficulty_low: _this.difficulty, difficulty_high: _this.difficulty, frequency: _this.frequency}).then(response => {
          let receivedData = response.body
          _this.homograph = receivedData.altered_string
          _this.originalText = receivedData.string
          _this.homographData = receivedData.homographs
          _this.selected = []
          _this.matches = []
          console.log(receivedData.homographs)
        })
      },
      homographClick (index, e) {
        if (this.selected.includes(index)) {
          let selectedIndex = this.selected.indexOf(index)
          this.selected.splice(selectedIndex, 1)
          e.target.parentElement.style.backgroundColor = null
        } else {
          e.target.parentElement.style.backgroundColor = '#FFEBEE'
          this.selected.push(index)
          this.checkMatch(index)
          this.selected = _.sortBy(this.selected, function (o) { return o })
        }
      },
      checkMatch (index) {
        console.log(index)
        let found = _.findIndex(this.homographData, function (o) {
          console.log(o.word_index)
          console.log(index)
          return o.word_index === index
        })
        console.log(found)
        if (found > 0) {
          this.matches.push({
            word: this.homograph[index],
            index: this.homographData[found].character_index
          })
        }
        console.log(this.matches)
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
  button.homograph.mu-flat-button {
    min-width: 0;
    .mu-flat-button-wrapper {
      .mu-flat-button-label {
        font-family: 'arial-unicode', sans-serif;
        text-decoration: none;
        text-transform: none;
        padding: 0 5px;
      }
    }
  }
  .sequence {
    margin-top: 15px;
  }
  .controls {
    .mu-raised-button {
      margin: 0 5px;
    }
    margin-bottom: 10px;
  }
</style>
