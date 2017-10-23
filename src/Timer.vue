<template>

	<div id="status">
		<p>
			{{ message }}{{ remaining }}
		</p>
	</div>
	
</template>


<script>

export default {
	name: 'timer',

	data () {
	  return {
	  	message: 'Click the start button to begin!',
	  	remaining: '',
	  	intervalId: null,
	  	// timerLength:
	  }
	},

	created() {
		this.$bus.$on('stateChange', ($event, timerLength) => {
			switch($event) {
				case 'capturing':
					this.startTimer(timerLength);
					break;
				case 'playing':
					this.stopTimer('Watch closely!');
					break;
				case 'processing':
					this.stopTimer();
					break;
				case 'gameover':
					this.stopTimer('Oops! Click the start button to play again.');
					break;
				default:
					console.log('event state: ', $event);
					break;
			}
		});
	},

	methods: {
		startTimer: function(timerLength) {
			if (this.intervalId === null) {
				this.message = '';
				this.remaining = timerLength;
				// Calling the tick function every 1000 miliseconds, but it won't stop until we tell it to
				this.intervalId = window.setInterval(this.tick, 1000);
			}
		},

		stopTimer: function(text) {
			window.clearInterval(this.intervalId);
			this.intervalId = null;
			this.remaining = '';
			if (text) {
				this.message = text;
			} else {
				this.message = '';
			}
		},

		tick: function() {
			console.log('tick');
			this.remaining -= 1;
			if (this.remaining === 0) {
				this.$bus.$emit('expired');
				window.clearInterval(this.intervalId);
				this.intervalId = null;
			}
		}
	}
}
	
</script>


<style>

#status {
	color: #999;
	font-weight: bold;
	height: 40px;
}
	
</style>