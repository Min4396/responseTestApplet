<template>
	<view class="background" @click="startPlay" v-bind:style="{ 'background-color': background_color}">
		
		<view v-if="start_flag===false" class="init">
			<view class="icon">
				<canvas canvas-id="Canvas-init"></canvas>
			</view>
			<view class="content">
				<view>锻炼逆向思维</view>
				<view>成为极少数的成功者</view>
				<text>单击任意位置开始</text>
			</view>
		</view>
		<view v-else class="startPlay_box">
			<block v-if="answer_flag===true">
				<canvas canvas-id="Canvas-startPlay"></canvas>
				<view class="time-schedule">
					<view class="sum-schedule" ></view>
					<view class="current-schedule" v-bind:style="{ width: currentSchedule + 'rpx'}"></view>
				</view>
				<view class="time-num">倒计时：{{time}}s</view>
				<text>{{showShape}}</text>
				<view class="buttons">
					<view @click="judgeIsCorrect(true)" class="yes_btn">√</view>
					<view @click="judgeIsCorrect(false)" class="no_btn">×</view>
				</view>
			</block>
			<block v-else>
				<block v-if="result===true">
					<view class="result">
						<view style="color: #007AFF">回答正确</view>
						<view>您已成功坚持{{qualified}}轮</view>
						<button class="continue" @click="continuePlay">点击继续</button>
					</view>
				</block>
				<block v-else>
					<view class="result">
						<view style="color: #ff557f">回答错误</view>
						<view>您成功坚持{{qualified}}轮</view>
						<view class="buttons">
							<button @click="resurrectionPlay" class="resurrection" type="default">复活继续({{freeResurrection}})</button>
							<button @click="resetPlay" class="reset" type="default">重新开始</button>
						</view>
					</view>
				</block>
			</block>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				start_flag: false, // 开始标记
				answer_flag: false,  // 答题标记
				background_color: 'rgb(170, 170, 255)',
				canvasWidth: 350,
				canvasHeight: 350,
				time: 5,  // 答题时间
				timer: null, // 计时器
				currentSchedule: 500,   // 进度条长度
				shapeList: ['Square', 'Circle', 'Rectangle', 'RegularPolygon'],
				colorList: ['white', 'red', 'blue', 'green', 'purple', 'yellow'],
				chineseShapeList: ['正方形', '圆形', '长方形', '多边形'],
				chineseColorList: ['白色', '红色', '蓝色', '绿色', '紫色', '黄色'],
				currentShape: null,  // 真实的图形
				showShape: null,     // 用户看到的题目
				pen: uni.createCanvasContext('Canvas-startPlay'),    // 画笔
				result: null,   // 用户结果正确与否
				freeResurrection: null,
				qualified: 0
			};
		},
		onLoad() {
			// 获取免费复活次数
			var data = uni.getStorageSync('freeResurrection');
			this.freeResurrection = data.three_num;
		},
		onReady: function (e) {
		    this.iconInit();
		},
		methods:{
			// 页面图标初始化
			iconInit: function(){
				var pen = uni.createCanvasContext('Canvas-init');
				
				pen.beginPath();
				pen.setStrokeStyle("#ffaa7f");
				pen.setLineWidth(2);
				pen.arc(75, 75, 60, 0, 2 * Math.PI);
				pen.stroke();
				pen.closePath();
				
				
				pen.beginPath();
				pen.moveTo(95, 75);
				pen.setStrokeStyle("#ffaaff");
				pen.arc(75, 75, 20, 0, 2 * Math.PI);
				pen.stroke();
				pen.closePath()
				
				pen.beginPath();
				pen.setStrokeStyle('#ffffff');
				pen.moveTo(75, 15);
				pen.lineTo(20, 97);
				pen.lineTo(130, 97);
				pen.lineTo(75, 15);
				pen.stroke();
				pen.closePath();
				
				pen.draw();
			},
			
			startPlay: function(){
				if(this.start_flag===false){
					this.start_flag = true;
					this.answer_flag = true;
					this.Timer();   // 开始倒计时
					this.pen.clearRect(0, 0, this.canvasWidth, this.canvasHeight);   // 清理画布
					var shapeDetailList = this.produceShape();   // 随机生成图形
					var color = shapeDetailList[0];
					var shape = shapeDetailList[1];
					var englishColor = shapeDetailList[2];
					var englishShape = shapeDetailList[3];
					this.currentShape = color + '的' + shape;   // 当前正确的图形
					// 根据形状与颜色绘制图形
					switch(shape){
						case '正方形': 
							this.drawSquare(englishColor);
							break;
						case '圆形': 
							this.drawCircle(englishColor);
							break;
						case '长方形': 
							this.drawRectangle(englishColor);
							break;
						case '多边形': 
							this.drawRegularPolygon(englishColor);
					}
					// 随机生成正确或错误的题目
					if(Math.random() < 0.55){   // 理论上错误的题目 
						shapeDetailList = this.produceShape();
						this.showShape = shapeDetailList[0] + '的' + shapeDetailList[1];
					}
					else{   // 正确的题目
						this.showShape = this.currentShape;
					}
				}
			},
			
			// 计时器
			Timer: function(){
				this.timer = setInterval(()=>{
					var time = this.time * 1000 - 100;
					this.time = (time / 1000).toFixed(1);
					this.currentSchedule = this.currentSchedule - 10;
					if(this.currentSchedule === 0){   // 超时
						clearInterval(this.timer);
						this.answer_flag = false;
						this.result = false;
						this.background_color = 'rgb(243, 223, 187)';
					}
				}, 100);
			},
			
			// 随机生成图形
			produceShape: function(){
				var shapeIndex = Math.floor(Math.random() * this.chineseShapeList.length);
				var shape = this.chineseShapeList[shapeIndex];  // 选取形状
				var colorIndex = Math.floor(Math.random() * this.chineseColorList.length);
				var color = this.chineseColorList[colorIndex];  // 选取颜色
				var englishShape = this.shapeList[shapeIndex];
				var englishColor = this.colorList[colorIndex];
				return [color, shape, englishColor, englishShape];
			},
			
			// 判断正确与否
			judgeIsCorrect: function(userChoose){
				this.answer_flag = false;
				this.background_color = 'rgb(243, 229, 230)';
				clearInterval(this.timer);  // 停止倒计时
				if((this.currentShape === this.showShape) === userChoose){
					this.result = true;
					this.qualified++;
				}
				else{
					this.result = false;
				}
			},
			
			// 继续游戏
			continuePlay: function(){
				this.time = 5;
				this.start_flag = false;
				this.currentSchedule = 500;
				this.background_color = 'rgb(170, 170, 255)';
				this.currentShape = null;
				this.showShape = null;
				this.result = null;
				this.pen = uni.createCanvasContext('Canvas-startPlay');
				this.startPlay();
			},
			
			// 重新开始
			resetPlay: function(){
				this.time = 5;
				this.timer = null;
				this.start_flag = false;
				this.currentSchedule = 500;
				this.background_color = 'rgb(170, 170, 255)';
				this.currentShape = null;
				this.showShape = null;
				this.result = null;
				this.pen = uni.createCanvasContext('Canvas-startPlay');
				this.qualified = 0;
			},
			
			// 圆形
			drawCircle: function(color){
				this.pen.setStrokeStyle("black");
				this.pen.setLineWidth(2);
				this.pen.arc(85, 90, 70, 0, 2 * Math.PI);
				this.pen.setFillStyle(color);
				this.pen.fill();
				this.pen.stroke();
				this.pen.draw()
			},
			
			// 长方形
			drawRectangle: function(color){
				this.pen.rect(10, 45, 155, 85);
				this.pen.setFillStyle(color);
				this.pen.fill();
				this.pen.stroke();
				this.pen.draw();
			},
			
			// 正方形
			drawSquare: function(color){
				this.pen.rect(20, 20, 135, 135);
				this.pen.setFillStyle(color);
				this.pen.fill();
				this.pen.stroke();
				this.pen.draw();
			},
			
			// 三角形
			drawTriangle: function(color){
				this.pen.moveTo(90, 30);
				this.pen.lineTo(10, 90);
				this.pen.lineTo(165, 90);
				this.pen.lineTo(90, 30);
				this.pen.setFillStyle(color);
				this.pen.fill();
				this.pen.stroke();
				this.pen.draw();
			},
			
			// 正多边形
			drawRegularPolygon: function(color){
				var num = Math.ceil(Math.random() * 6) + 3;
				this.createPolygonPath(this.canvasWidth/4,this.canvasHeight/4,this.canvasWidth/6,num,0);
				this.pen.setFillStyle(color);
				this.pen.fill();
				this.pen.stroke();
				this.pen.draw();
			},
			
			// 连接正多边形的点
			createPolygonPath: function(centerX,centerY,radius,sides,startAngle) {
			    let points = this.getPolygonPoints(centerX,centerY,radius,sides,startAngle);
			
			    this.pen.beginPath();
			    this.pen.moveTo(points[0].x,points[0].y);
			
			    for (let i=0;i<sides;++i){
			        this.pen.lineTo(points[i].x,points[i].y);
			    }
			    this.pen.closePath();
			},
			
			// 得到正多边形的点
			getPolygonPoints: function(centerX,centerY,radius,sides,startAngle) {
				class Point{
				    constructor(x, y) {
				        this.x = x;
				        this.y = y;
				    }
				};
			    let points = [],
			        //这里的angle是基于钟表0点的位置开始计算，0点位置为0度，3点位置为π/2度
			        angle = startAngle||0;
			
			    for (let i=0;i<sides;++i){
			        points.push(new Point(centerX+radius*Math.sin(angle),centerY+radius*Math.cos(angle)));
			        angle += 2*Math.PI/sides;
			    }
			
			    return points;
			}
		}
	}
</script>

<style lang="less">
	
*{
	margin: 0;
	padding: 0;
}

.background{
	width: 750rpx;
	height: 1400rpx;
	background-color: #aaaaff;
}

.init{
	position: fixed;
	top: 15%;
	left: 10%;
	width: 600rpx;
	height: 950rpx;
	.icon{
		position: relative;
		left: 25%;
		width: 300rpx;
		height: 300rpx;
	}
	.content{
		margin-top: 70rpx;
		font-size: 40rpx;
		text-align: center;
		color: white;
		view{
			margin-top: 30rpx;
		}
		text{
			position: absolute;
			top: 95%;
			left: 23%;
		}
	}
}

.startPlay_box{
	display: flex;
	justify-content: center;
	canvas{
		position: absolute;
		top: 10%;
		left: 27%;
		width: 350rpx;
		height: 350rpx;
	}
	.time-schedule{
		position: relative;
		top: 500rpx;
		.sum-schedule{
			width: 500rpx;
			height: 20rpx;
			background-color: white;
			border-radius: 20rpx;
			margin: 0rpx 50rpx;
		}
		.current-schedule{
			position: relative;
			top: -20rpx;
			height: 20rpx;
			background-color: #ffaa00;
			border-radius: 20rpx;
			margin: 0rpx 50rpx;
		}
	}
	.time-num{
		position: absolute;
		top: 550rpx;
		// left: -100rpx;
		display: flex;
		justify-content: center;
	}
	text{
		position: absolute;
		top: 55%;
		font-size: 40rpx;
		font-weight: bold;
		color: white;
	}
	.buttons{
		display: flex;
		justify-content: space-around;
		position: absolute;
		top: 65%;
		width: 750rpx;
		view{
			width: 100rpx;
			height: 100rpx;
			font-size: 60rpx;
			border-radius: 50%;
			display: flex;
			justify-content: center;
			align-items: center;
			border: 5rpx solid white;
		}
		.yes_btn{
			background-color: #00ff00;
		}
		.no_btn{
			background-color: #ff557f;
		}
	}
}

.result{
	position: absolute;
	top: 30%;
	display: flex;
	justify-content: center;
	align-items: center;
	flex-direction: column;
	view:first-of-type{
		font-size: 70rpx;
	}
	view:nth-of-type(2){
		margin-top: 30rpx;
		font-size: 40rpx;
	}
	button{
		width: 250rpx;
		height: 100rpx;
		font-size: 35rpx;
		border: 3px solid white;
		color: white;
	}
	.continue{
		margin-top: 200rpx;
		background-color: #55ffff;
	}
	.buttons{
		position: absolute;
		top: 400rpx;
		left: -230rpx;
		display: flex;
		justify-content: space-between;
		.resurrection{
			width: 250rpx;
			background-color: #87f70e;
		}
		.reset{
			width: 220rpx;
			background-color: #f7dd48;
		}
	}
}

</style>
