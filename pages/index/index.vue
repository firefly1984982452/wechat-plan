<template>
	<view class="content">
		<view class="line">
			<text class="text">今天是今年的进度的:</text>
			<text class="bold">{{ todayPlan }}%</text>
		</view>
		<view class="plan"><i :style="'width:' + todayPlan + '%'"></i></view>

		<view class="line">
			<text class="text">你的生日是:</text>
			<e-picker mode="date" class="bold" @change="change">{{ birthDay || '选择出生日期' }}</e-picker>
		</view>

		<view class="line" v-show="birthDay">
			<text class="text">你的年龄是:</text>
			<text class="bold">{{ age }}岁</text>
		</view>

		<view class="line" v-show="birthDay">
			<text class="text">对于80岁，你目前的进度是:</text>
			<text class="bold">{{ agePlan }}%</text>
		</view>
		<view class="plan"><i :style="'width:' + agePlan + '%'"></i></view>

		<view class="line" v-show="birthDay">
			<text class="text">你目前的年龄相当于时钟上的:</text>
			<text class="bold">{{ timePlan }}</text>
		</view>

		<view class="clock-content">
			<div class="clock">
				<div class="hand hour" :style="'transform: rotate(' + hourDeg + 'deg);'"></div>
				<div class="hand min" :style="'transform: rotate(' + minDeg + 'deg);'"></div>
			</div>
		</view>

		<view class="line">
			<text class="bold">{{ peom }}</text>
			<button class="btn-change" @click="like">收藏</button>
			<button class="btn-change" @click="getPoem">换一句</button>
		</view>

		<view class="poem-list">
			<text v-for="(item, index) in peomList" :key="index">{{ item }}</text>
		</view>

		<button open-type="share"></button>
	</view>
</template>

<script>
const poem = require('../../utils/jinrishici');
export default {
	data() {
		return {
			birthDay: '',
			todayPlan: '',
			agePlan: '',
			age: 0,
			timePlan: '',
			birthDayKey: 'birthDay',
			hourDeg: 0,
			minDeg: 0,
			peom: '',
			peomList: [],
			peomKey: 'peomKey'
		}
	},
	onLoad() {
		this.getPoem();
		this.getPoemList();
		this.init();
		this.getPlan();
	},
	onShareAppMessage(res) {
		if (res.from === 'button') {
			console.log("来自页面内按钮分享")
		}
		return {
			path: "",
			title: "分享"
		}
	},
	methods: {
		getPoem() {
			poem.load(res => {
				console.log(res.data.content)
				this.peom = res.data.content;
			})
		},
		init() {
			uni.getStorage({
				key: this.birthDayKey,
				success: res => {
					console.log(res)
					this.birthDay = res.data;
					this.getAgePlan();
					this.getAge24Time();
				},
				fail: () => {
					uni.setStorage({
						key: this.birthDayKey,
						data: '',
						success: res => {
							console.log('初始化成功')
						}
					})
				}
			})
		},
		change(e) {
			this.birthDay = e;
			// 本地存储
			uni.setStorage({
				key: this.birthDayKey,
				data: this.birthDay,
				success: res => {
					this.getAgePlan();
					this.getAge24Time();
				}
			})
		},
		// 计算今天是今年的进度的多少
		getPlan() {
			let now = new Date();
			let year = now.getFullYear();
			let yearStartTimeStamp = Math.floor(new Date(year + '/01/01 00:00:00').getTime() / 1000);
			let yearEndTimeStamp = Math.floor(new Date(year + '/12/31 23:59:59').getTime() / 1000);
			let nowTimeStamp = Math.floor(now.getTime() / 1000);
			let reduceTotal = yearEndTimeStamp - yearStartTimeStamp;
			let reduceNow = nowTimeStamp - yearStartTimeStamp;
			let plan = parseFloat(((reduceNow / reduceTotal) * 100).toFixed(2));
			this.todayPlan = plan;
		},
		// 对于80岁，你目前的进度是多少
		getAgePlan() {
			console.log(this.birthDay)
			let now = new Date();
			let str = this.birthDay;
			let year = str.substr(0, 4);
			let month = str.substr(5, 2);
			let day = str.substr(8, 4);
			this.age = now.getFullYear() - year;
			let startBirthDay = Math.floor(new Date(year + '/' + month + '/' + day + ' 00:00:00').getTime() / 1000);
			let endBirthDay = Math.floor(new Date(Number(Number(year) + 80) + '/' + month + '/' + day + ' 23:59:59').getTime() / 1000);
			let nowTimeStamp = Math.floor(now.getTime() / 1000);
			let reduceTotal = endBirthDay - startBirthDay;
			let reduceNow = nowTimeStamp - startBirthDay;
			let plan = parseFloat(((reduceNow / reduceTotal) * 100).toFixed(2));
			this.agePlan = plan;
		},
		// 你目前的年龄相当于时钟上的几点几分
		getAge24Time() {
			console.log('>>>')
			let minutesTotal = 24 * 60;
			let agePlan = Math.ceil(minutesTotal * (this.agePlan / 100));
			let hour = Math.trunc(agePlan / 60);
			let minute = Number(agePlan % 60);
			this.timePlan = hour + '点' + minute + '分';
			console.log(agePlan, this.timePlan);
			this.minDeg = Number((minute / 60) * 360 + (60 / 60) * 6 + 90);
			this.hourDeg = Number((hour / 12) * 360 + (minute / 60) * 30 + 90);
		},
		like() {
			this.peomList.push(this.peom);
			uni.setStorage({
				key: this.peomKey,
				data: this.peomList,
				success: res => {
					uni.showModal({
						title: '收藏成功'
					})
				}
			})
		},
		getPoemList() {
			uni.getStorage({
				key: this.peomKey,
				success: res => {
					this.peomList = res.data;
				},
				fail: () => {
					this.peomList = [];
				}
			})
		}
	}
}
</script>

<style>
.content {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	font-size: 30rpx;
	padding: 10rpx;
}

.line {
	display: flex;
	flex-direction: row;
	width: 100%;
	margin: 25rpx 10rpx 10rpx;
}

.text {
	display: flex;
	flex: 2;
}

.bold {
	display: flex;
	flex: 1;
	font-size: 35rpx;
	color: #5d97fb;
	font-weight: bold;
}

.plan {
	width: 90%;
	height: 15rpx;
	background: #5570a3;
	border-radius: 5rpx;
	position: relative;
	margin: 0 0 20rpx;
}

.plan i {
	position: absolute;
	left: 0;
	top: 0;
	height: 15rpx;
	background-color: #5d97fb;
}

.clock-content {
	width: 500rpx;
	margin: 0 auto;
	height: 500rpx;
	display: flex;
	justify-content: center;
	align-items: center;
}

.clock {
	width: 300rpx;
	height: 300rpx;
	border: 5rpx solid #5570a3;
	border-radius: 50%;
	position: relative;
	padding: 2rpx;
	box-shadow: 0 0 0 4rpx rgba(0, 0, 0, 0.1), inset 0 0 0 3rpx #efefef,
		inset 0 0 10rpx #333, 0 0 10rpx rgba(0, 0, 0, 0.2);
}

.btn-change {
	width: 200rpx;
	height: 50rpx;
	line-height: 50rpx;
	font-size: 25rpx;
}


.hand {
	width: 45%;
	height: 6rpx;
	background: #5570a3;
	position: absolute;
	top: 50%;
	transform-origin: 100%;
	transform: rotate(90deg);
	transition: all 0.05s;
	transition-timing-function: cubic-bezier(0.1, 2.7, 0.58, 1);
}

.hand.sec {
	height: 2rpx;
}

.hand.hour {
	width: 30%;
	left: 16%;
	top: 50%;
}

.poem-list text {
	display: block;
	text-align: left;
	width: 750rpx;
}</style>
