<template>
  <div id="app">

    <h1>Vuemon</h1>

    <div id="simon">

      <timer></timer>

      <div class="row">
        <div id="green" class="light col noSelect" v-bind:class="{ 'bright': currentButton === 'green' }" v-on:click="captureTap('green')"></div>
        <div id="red" class="light col noSelect" v-bind:class="{ 'bright': currentButton === 'red' }" v-on:click="captureTap('red')"></div>
      </div>

      <div class="row">
        <div id="yellow" class="light col noSelect" v-bind:class="{ 'bright': currentButton === 'yellow' }" v-on:click="captureTap('yellow')"></div>
        <div id="blue" class="light col noSelect" v-bind:class="{ 'bright': currentButton === 'blue' }" v-on:click="captureTap('blue')"></div>
      </div>

    </div>

    <div id="controls" class="row">
      <div class="col"><button class="start" v-on:click="start">Start</button></div>
    </div>

    <div id="history">
      <p><strong>Current Sequence:</strong> {{ current }}</p>
      <p><strong>Longest Sequence:</strong> {{ longest }}</p>
    </div>

  </div>
</template>

<script>
import timer from './Timer.vue'

export default {
  name: 'app',
  components: {
    timer
  },

  data () {
    return {
      longest: 0,
      sequence: [],
      playSequenceCounter: 0,
      playSequenceId: null,
      taps: [],
      lights: [ 'red', 'green', 'yellow', 'blue' ],
      timerIsActive: false,
      timerLength: 5,
      currentButton: '',
    }
  },

  computed: {
    current: function() {
      return this.sequence.length;
    },

    tapsMatchesSequence: function() {
      let partialSequence = this.sequence.slice(0, this.taps.length);
      // Compare the two arrays to see if they match (be careful... this is kind of a cheating way)
      // This only works because I have total control of the two arrays
      return (JSON.stringify(this.taps) == JSON.stringify(partialSequence));
    }
  },

  created() {
    this.$bus.$on('expired', ($event) => {
      this.gameOver();
    });
  },

  mounted() {

  },
  methods: {

    changeState: function(newState) {
      switch(newState) {
        case 'ready':
          this.timerIsActive = false;
          this.$bus.$emit('stateChange', 'ready');
          break;
        case 'playing':
          this.timerIsActive = false;
          this.$bus.$emit('stateChange', 'playing');
          break;
        case 'capturing':
          this.timerIsActive = true;
          this.$bus.$emit('stateChange', 'capturing', this.timerLength);
          break;
        case 'processing':
          this.timerIsActive = false;
          this.$bus.$emit('stateChange', 'processing');
          break;
        case 'gameover':
          this.timerIsActive = false;
          this.$bus.$emit('stateChange', 'gameover');
          break;
        default:
          console.log('changeState: event state unknown', $event);
      }
    },

    start: function() {
      this.sequence = [];
      this.addToSequence();
      this.playSequence();
    },

    chooseRandomLight: function() {
      var index = Math.floor(Math.random() * 4);
      console.log(index);
      return this.lights[index];
    },

    addToSequence: function() {
      // adds one more step to the sequence
      this.sequence.push(this.chooseRandomLight());
      console.log('sequence: ', this.sequence);
    },

    playSequence: function() {
      // `setInterval` sets something up to occur, `setTimeout` sets a timeout
      this.playSequenceId = window.setInterval( () => {

        this.changeState('playing');
        this.currentButton = '';

        window.setTimeout ( () => {
          this.currentButton = this.sequence[ this.playSequenceCounter++ ];
          if (this.playSequenceCounter > this.sequence.length) {

            // We're done!
            window.clearInterval(this.playSequenceId);
            this.playSequenceId = null;
            this.playSequenceCounter = 0;
            this.changeState('capturing');
          }

        }, 300);

      }, 600);
      
    },

    captureTap: function(color) {
      console.log('captureTap: color: ', color);

      if (this.timerIsActive === true) {
        this.currentButton = color;

        window.setTimeout(() => {
          this.currentButton = '';
        }, 300);

        this.changeState('processing');

        this.taps.push(color);

        // Checking to see if each tap is correct
        if(this.tapsMatchesSequence == true) {

          if(this.taps.length === this.sequence.length) {
            // Matches completely
            this.taps = [];
            this.addToSequence();
            this.playSequence();
            this.changeState('capturing');
          } else {
            // Matches so far
            this.changeState('capturing');
          }

        } else {
          // Taps does not match sequence
          this.gameOver();
        }

      } else {
        // ignore the tap
      }
    },

    gameOver: function() {
      console.log('Game over!');
      this.changeState('gameover');
      this.taps = [];

      if (this.longest < this.sequenceLength) {
        this.longest = this.sequenceLength;
      }
    },

  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

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

#simon {
  padding: 20px;
}

#controls {
  padding-bottom: 20px;
}

#status {
  padding-bottom: 20px;
}

.row {

}

.col {
  display: inline-block;
}

.start {
  padding: 10px;
  font-size: 18px;
}

.light {
  margin: 20px;
}

#red {
  height: 100px;
  width: 100px;
  background: #E53A40;
  opacity: 0.1;
}

#yellow {
  height: 100px;
  width: 100px;
  background: #EFDC05;
  opacity: 0.1;
}

#green {
  height: 100px;
  width: 100px;
  background: #75D701;
  opacity: 0.1;
}

#blue {
  height: 100px;
  width: 100px;
  background: #30A9DE;
  opacity: 0.1;
}

.bright {
  opacity: 1.0 !important;
}

.noSelect {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

</style>
