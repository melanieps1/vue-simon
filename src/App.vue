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
      taps: [],
      lights: [ 'red', 'green', 'yellow', 'blue' ],
      timerIsActive: false,
      timerLength: 10,
      currentButton: '',
    }
  },

  computed: {
    current: function() {
      return this.sequence.length;
    }
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
      this.sequence.push(this.chooseRandomLight());
    },

    playSequence: function() {
      this.changeState('playing');
      this.changeState('capturing');
    },

    captureTap: function(color) {
      console.log('captureTap: color: ', color);
      if (this.timerIsActive === true) {
        this.currentButton = color;
        window.setTimeout(() => {
          this.currentButton = '';
        }, 300);
      } else {
        // ignore the tap
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
  border: 1px solid #000;
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
