<template>
	<canvas id="progress-bar-01" width="375" height="50" @click="clickFunc"></canvas>
</template>

<script>
	export default {
		name : "Progress",
		props : {
			duration : Number,
			seekTime : Function,
			status : String,
			trackid : Number,
			currentTimeTrack : Number,
			errProgress : Boolean
		},
		data(){
			return {
				// status
				progressStatus : "pause",
				//

				progressCanvas : null,
				ctx : null,
				width : null,
				height : null,
				posY_durationTimeText : 20,
				last_time : new Date().getTime(),
				last_time2 : new Date().getTime(),
				RAF : false,

				// progress line
				progress : {
					height : 15,
					posX: 0,
					posY: null,
					stepTimeUpdate : 500,
					colorProgress : "#dfe4ea",
					time : 0,
					currentX : null
				},
				//

				// time
				time : {
					durationTime : {
						posX : null,
						posY : 20,
						text : "00:00",
					},
					currentTime : {
						posX : 20,
						posY : 20,
						text : "00:00",
						time : null,
					},
					color : "#fff"
				}
			}
		},

		watch : {
			trackid : function(newVal, oldVal){
				// if (!this.errProgress) {
				// set time duration
				this.time.durationTime.text = this.millisecondsToMin(this.$props.duration);
				// init
				this.init();
				// update progress , draw ,...
				this.update();
			// }
			},

			status : function(newVal, oldVal){
				// gan progressStatus by status PlayView
				this.progressStatus = newVal;
				// set time duration
				this.time.durationTime.text = this.millisecondsToMin(this.$props.duration);

				if(this.progressStatus === "pause") { // pause or unpause
					this.pauseProgress();
				} else {
					this.unpauseProgress();
				}
			},

			currentTimeTrack : function(newVal, oldVal){
				// if(newVal) {}
				console.log(newVal)
				this.time.currentTime.time = newVal;
				this.drawFunction();
			}
		},

		methods : {
			millisecondsToMin(mill){
				let arr = [];

				let hms = function(mill){
					return {
						hours : Math.floor(mill/(60*60*1000)),
						min : Math.floor((mill/60000)%60),
						sec : Math.floor((mill/1000)%60)
					}
				}(mill);

				if (hms.hours > 0) {
					arr.push(hms.hours)
				}
				arr.push( (hms.min < 10?"0"+hms.min:hms.min) );
				arr.push( (hms.sec <10? "0"+hms.sec:hms.sec) );
				return arr.join(":");
			},

			init(){
				console.log('init')
				this.time.currentTime.time = 0;
				this.time.currentTime.text = this.millisecondsToMin(this.time.currentTime.time);
				this.progress.time = 0;
				this.progress.currentX = 0;
				this.RAF = true;

				// swap tab support
				// document.addEventListener("visibilitychange", function(){
				// 	// if (!document.hidden && this.props.musicAppState.searchReducer.track_currentTime) {
				// 	// 	this.currentTimeProgress= this.currentTime = this.props.musicAppState.searchReducer.track_currentTime;

				// 	// 	console.log(this.currentTime + "day la currentTime");
				// 	// 	// self.timePlayed = new Date().getTime() - self.currentTime;
				// 	// 	// this.progressX_current = (this.currentTime/this.duration)*this.width
				// 	// }

				// }.bind(this), false);
				// this.progress.time = 
			},

			update(){
				let time_now = new Date().getTime();			
				if (time_now - this.last_time >=200) {
					this.ctx.clearRect(0, 0, this.width, this.height);
					this.progress.time+= 200;

					if (time_now - this.last_time2 >=1000) {
						this.time.currentTime.time += 1000;
						this.time.currentTime.text = this.millisecondsToMin(this.time.currentTime.time);
						this.last_time2 = time_now;
					}

					this.drawFunction();
					this.last_time = time_now;
				}

				if (this.time.currentTime.time >= this.$props.duration - 1000) { // finish track
					this.time.currentTime.time = this.$props.duration;
					// this.curTimeText = this.millisecondsToMin(this.time.currentTime.time);
					this.drawFunction();
					console.log('het bai hat');
					// this.pauseProgress();
					// this.props.progressStatus("finish");
				}

				if (this.RAF) {
					this.myAnimation = window.requestAnimationFrame(this.update);
				}
			},

			drawFunction(){
				// let width = this.width < 375 ? this.width : 375;
				this.progress.currentX = (this.progress.time/this.$props.duration)*this.width;
				this.ctx.beginPath();
				this.ctx.fillStyle = this.progress.colorProgress;
				this.ctx.fillRect(this.progress.posX, this.progress.posY, this.progress.currentX, this.progress.height);
				
				// draw current time
				this.ctx.beginPath();
				this.ctx.textAlign="left";
				this.ctx.font="16px Verdana";
				this.ctx.fillStyle = this.time.color;
				this.ctx.fillText(this.time.currentTime.text, this.time.currentTime.posX, this.time.currentTime.posY);
				

				// draw duration time
				this.ctx.beginPath();
				this.ctx.textAlign="right";
				this.ctx.fillText(this.time.durationTime.text, this.time.durationTime.posX, this.time.durationTime.posY);
			},

			pauseProgress(){
				this.RAF = false;
			},

			unpauseProgress(){
				this.RAF = true;
				this.update();
			},

			clickFunc(e){
				if (this.RAF) {
					let x = e.pageX;
					let seekTime = (x/this.width)*this.$props.duration;
					this.time.currentTime.time = seekTime;
					this.progress.time = seekTime;
					this.$props.seekTime(seekTime);
				}
			}
		},

		mounted(){
			var vendors = ['ms', 'moz', 'webkit', 'o'];

			for(var x = 0; x < vendors.length && !window.requestAnimationFrame; ++x) {

			    window.requestAnimationFrame = window[vendors[x]+'RequestAnimationFrame'];

			    window.cancelAnimationFrame = window[vendors[x]+'CancelAnimationFrame'] ||

			    window[vendors[x]+'CancelRequestAnimationFrame'];
			}

			this.progressCanvas = document.getElementById('progress-bar-01');
			this.ctx = this.progressCanvas.getContext('2d');
			this.width = this.progressCanvas.width;
			this.height = this.progressCanvas.height;

			this.time.durationTime.posX = this.width - 20;
			this.progress.posY = this.height - this.progress.height;
		}
	}
</script>