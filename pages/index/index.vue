<template>
	<view class="content">
		<view class="line">
			<text class="text">今天是今年的进度的:</text>
			<text class="bold">{{todayPlan}}%</text>
		</view>
		<view class="plan"><i :style="'width:'+todayPlan+'%'"></i></view>
		<view class="line">
			<text class="text">你的生日是:</text>
			<e-picker mode="date"  class="bold"  @change="change">{{birthDay||'选择出生日期'}}</e-picker>
		</view>
		<view class="line" v-show="birthDay">
			<text class="text">对于80岁，你目前的进度是:</text>
			<text class="bold">{{agePlan}}%</text>
		</view>
		<view class="plan"><i :style="'width:'+agePlan+'%'"></i></view>
		<view class="line" v-show="birthDay">
			<text class="text">你目前的年龄相当于时钟上的:</text>
			<text class="bold">{{timePlan}}</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				birthDay: '',
				todayPlan:'',
				agePlan:'',
				timePlan:'',
				birthDayKey: 'birthDay'
			}
		},
		onLoad() {
			this.init();
			this.getPlan();
			this.getAgePlan();
			this.getAge24Time();
		},
		methods: {
			init(){
				uni.getStorage({
					key: this.birthDayKey,
					success: res => {
						this.birthDay = res.data;
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
				})
				this.getAgePlan();
				this.getAge24Time();
			},
			// 计算今天是今年的进度的多少
			getPlan(){
				let now = new Date();
				let year = now.getFullYear();
				let yearStartTimeStamp = Math.floor(new Date(year + '/01/01 00:00:00').getTime()/1000);
				let yearEndTimeStamp = Math.floor(new Date(year + '/12/31 23:59:59').getTime()/1000);
				let nowTimeStamp = Math.floor(now.getTime()/1000);
				let reduceTotal = yearEndTimeStamp - yearStartTimeStamp;
				let reduceNow = nowTimeStamp - yearStartTimeStamp;
				let plan = parseFloat(((reduceNow/reduceTotal)*100).toFixed(2));
				this.todayPlan = plan;
			},
			// 对于80岁，你目前的进度是多少
			getAgePlan () {
				console.log(this.birthDay)
				let now = new Date();
				let str = this.birthDay;
				let year = Number(str.substr(0,4));
				let month = Number(str.substr(5,2));
				let day = Number(str.substr(8,4));
				let startBirthDay = Math.floor(new Date(year + '-' + month + '-' + day + ' 00:00:00').getTime()/1000);
				let endBirthDay = Math.floor(new Date(Number(year + 80) + '-' + month + '-' + day + ' 23:59:59').getTime()/1000);
				let nowTimeStamp = Math.floor(now.getTime()/1000);
				let reduceTotal = endBirthDay - startBirthDay;
				let reduceNow = nowTimeStamp - startBirthDay;
				let plan = parseFloat(((reduceNow/reduceTotal)*100).toFixed(2));
				this.agePlan = plan;
				console.log(plan)
			},
			// 你目前的年龄相当于时钟上的几点几分
			getAge24Time() {
				let minutesTotal = 24*60;
				let agePlan = Math.ceil(minutesTotal * (this.agePlan/100));
				let hour = Math.trunc(agePlan/60);
				let minute = agePlan%60;
				this.timePlan = hour+'点'+minute+'分';
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
		font-size: 10rpx;
	}
	.line {
		display: flex;
		flex-direction: row;
		width: 100%;
		margin: 10rpx 10rpx 0;
	}
	.text {
		display: flex;
		flex: 2;
	}
	.bold {
		display: flex;
		flex: 1;
		font-size: 14rpx;
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
</style>
