<template>
	<canvas id="Background-Canvas" height="667" class="bg-01"></canvas>
</template>

<script>
	import Wave from './Wave.js';
	export default {
		name: "backgroundCanvas",

		methods : {
			
		},
		mounted() {
			var vendors = ['ms', 'moz', 'webkit', 'o'];

			for(var x = 0; x < vendors.length && !window.requestAnimationFrame; ++x) {

			    window.requestAnimationFrame = window[vendors[x]+'RequestAnimationFrame'];

			    window.cancelAnimationFrame = window[vendors[x]+'CancelAnimationFrame'] ||

			    window[vendors[x]+'CancelRequestAnimationFrame'];
			}

			let bg = {
				initFunc : function(){
					// get width window & some 
					let windowWidth = window.innerWidth;
					this.myCanvas = document.querySelector('#Background-Canvas'),
						this.ctx = this.myCanvas.getContext('2d');
					this.ctx.globalCompositeOperation = "source-over";
					this.myCanvas.width = windowWidth;
					//

					// define color for waves
					this.grd1=this.ctx.createLinearGradient(0,0,this.myCanvas.width/2,0);
					this.grd2=this.ctx.createLinearGradient(0,0,2*this.myCanvas.width/3,0);
					this.grd3=this.ctx.createLinearGradient(0,0,this.myCanvas.width/2,0);
					
					this.grd1.addColorStop(0,"#7f49a6");
					this.grd1.addColorStop(1,"#df4d7c");

					this.grd2.addColorStop(0,"#734297");
					this.grd2.addColorStop(1,"#c24472");

					this.grd3.addColorStop(0,"#563275");
					this.grd3.addColorStop(1,"#86315c");

					this.timeNow = this.timeExecute = new Date().getTime();
				},

				drawFunc : function(){
					this.timeNow = new Date().getTime();
					if (this.timeNow - this.timeExecute >=150) {
						this.ctx.clearRect(0, 0, this.myCanvas.width, this.myCanvas.height);
						this.c.draw();
						this.b.draw();
						this.wave1.draw();
						this.timeExecute = this.timeNow;
					};
					window.requestAnimationFrame(this.drawFunc);
					// console.log('hello');
				},

				start : function(){
					this.initFunc = this.initFunc.bind(this);
					this.drawFunc = this.drawFunc.bind(this);
					
					let t = new Promise((resolve, reject)=>{
						this.initFunc();
						this.wave1 = new Wave("1", 130, 35, 0.2, this.myCanvas, this.ctx, this.grd1);
						this.b = new Wave("2", 150, 40, 0.2, this.myCanvas, this.ctx, this.grd2);
						this.c = new Wave("3", 140, 35, 0.3, this.myCanvas, this.ctx, this.grd3);
						resolve();
					});

					t.then(()=>{
						this.drawFunc();
					})
				}
			}

			bg.start();
		}
	}
</script>