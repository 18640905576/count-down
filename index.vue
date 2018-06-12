<template>
	<div class="tg-countdown flex-container">
		<div class="countdown-container" :class="style+' '+key+' '+' '+spaceStyleList[spaceType]" v-for="(key,value) in contentList">
			<span>{{value}}</span>
		</div>
	</div>
</template>
<style>

	.countdown-container{
		background-color:#232323;
		color:#ffffff;
		padding:1px 1px;
		border-radius:2px;
		position: relative;
	}
	.countdown-container:after{
		color:#232323;
	}
	.white{
		background-color:#fff !important;
		color:#222222 !important;
	}
	.white:after{
		color:#fff !important;
	}
	.red{
		background-color:#ff4c48 !important;
		color:#fff !important;
	}
	.red:after{
		color:#ff4c48 !important;
	}
	.green{
		background-color:#16cc90 !important;
		color:#fff !important;
	}
	.green:after{
		color:#16cc90 !important;
	}
	.transparent{
		background-color: transparent;
		color:#222 !important;
	}
	.day{
		margin-right: 10px;
	}
	.day.symbol{
		margin-right: 20px;
	}
	.day:after{
		content:'';
		position: absolute;
		right:-10px;
		top:1px;
	}
	.day.word:after{
		content: '天';		
	}
	.day.symbol:after{
		content: '天:';
		right:-16px;
	}
	.hour,.minute,.sec{
		margin-right: 10px;
	}
	.hour.word{
		margin-right: 22px;
	}
	.hour:after,.minute:after,.sec:after{
		position: absolute;
		top:1px;
	}
	.hour.symbol:after,.minute.symbol:after{
		content: ':';
		right:-7px;
	}
	.sec.symbol:not(:last-child):after{
		content: '.';
		right:-7px;
	}
	.hour.word:after,.minute.word:after,.sec.word:after{
		content:'';
	}
	.hour.word:after{
		content: '小时';
		right:-22px;
	}
	.minute.word:after{
		content: '分';
		right:-10px;
	}
	.sec.word:after{
		content: '秒';
		right:-10px;
	}

	.timeBefore{
		color:#232323;
		position: absolute;
		left:-7px;
		top:0;
	}
</style>
<script>
	export default{
		props:{
			nowTime:{type:Number,default:+new Date()},
			endTime:{type:Number,default:0},
			style:{type:String,default:''}, //样式，不填为黑色背景
			spaceType:{type:Number,default:1}, 
			//时间之间的连接方式，{1：符号【x天:xx:xx:xx.x】，2：中文文字【x天xx小时xx分xx秒x】，3：不填【x xx xx xx x】}
			//默认颜色跟着背景颜色走，如果需要改则在页面中用css改
			format:{type:String,default:'hh:mm:ss'}, //'dd hh:mm:ss.ms'分别对应天 小时 分 秒 十分之一秒，每一个都为可选的，默认显示时分秒
			pad:{type:Boolean,default:true} // 是否将时、分、秒格式化两位，空位用0填充，默认为填充

		},
		data(){
			return {
				contentList:{day:'0', hour:'0',minute:'0',sec:'0',millSec:'0'},
				spaceStyleList : ['','symbol', 'word'],
				timer:null
			}
		},
		ready(){
			// TODO
			// 根据format，确认显示的样式
			// 根据spaceStyle，确认时间之间的连接方式
			// 根据是否显示十分之一秒，确认执行间隔时间
			// 如果结束时间比开始时间早，显示0
			var vm = this
				,format = this.format
				,intervalTime
				;

			vm.showDay = (/dd/.test(format));
			vm.showHour = (/hh/.test(format));
			vm.showMin = (/mm/.test(format));
			vm.showSecond = (/ss/.test(format));
			vm.showMilli = (/ms/.test(format));

			intervalTime = vm.showMilli?100:1000;
			if(this.endTime&&this.nowTime&&this.endTime>this.nowTime){
				vm.nowTime+=intervalTime;
				this.timer=setInterval(function(){
					vm.nowTime+=intervalTime;
				}, intervalTime);
			}else{
				this.contentList = {}
				if(this.showDay){this.contentList.day = '0'};
				if(this.showHour){this.contentList.hour = '0'};
				if(this.showMin){this.contentList.minute = '0'};
				if(this.showSecond){this.contentList.sec = '0'};
				if(this.showMilli){this.contentList.millSec = '0'};
			}
		},
		computed:{

		},
		watch:{
			nowTime(value){
				if(this.endTime&&this.nowTime){
					// 获取剩余时间
					// 如果剩余时间小于0，倒计时显示为0
					// 如果剩余时间大于0，进行逻辑展示 day hour minute sec millSec
					var countDownTime=this.endTime-value;
					if(countDownTime<0){
						clearTimeout(this.timer);
						this.contentList = {}
						if(this.showDay){this.contentList.day = '0'};
						if(this.showHour){this.contentList.hour = '0'};
						if(this.showMin){this.contentList.minute = '0'};
						if(this.showSecond){this.contentList.sec = '0'};
						if(this.showMilli){this.contentList.millSec = '0'};
						this.$dispatch('countDownEnd',this);
					}else{ 
						// day 86 400 000
						// hour 如果有天，则0~23，否则无上限
						// minute 如果有小时，则0~59，否则无上限
						// sec 如果有分钟，则0~59，否则无上限
						// millSec 如果有秒，则0~9，否则无上限
						
						var day = Math.floor((countDownTime / 3600000) / 24)
							,hour = Math.floor(this.showDay?(countDownTime / 3600000) % 24 : countDownTime / 3600000)
							,minute = Math.floor(this.showHour?(countDownTime / 60000) % 60 : countDownTime / 60000)
							,sec = Math.floor(this.showMin?(countDownTime / 1000) % 60 : countDownTime / 1000)
							,millSec = Math.floor(this.showSecond?(countDownTime / 100) % 10 : countDownTime / 100)
							,obj = {hour,minute,sec}
							,timeObj = []
							;
						// 格式化时、分、秒
						timeObj = this.pad?_.mapValues(obj,item=>_.padStart(item,2,0)):obj;
						// 全部时间的对象
						timeObj = _.merge({day : day}, timeObj, {millSec : millSec});
						// 根据配置选择如何显示
						this.contentList = {}
						if(this.showDay){this.contentList.day = timeObj.day};
						if(this.showHour){this.contentList.hour = timeObj.hour};
						if(this.showMin){this.contentList.minute = timeObj.minute};
						if(this.showSecond){this.contentList.sec = timeObj.sec};
						if(this.showMilli){this.contentList.millSec = timeObj.millSec};
					}
				}
			},
			endTime(value){
				var vm=this
					,intervalTime = this.showMilli?100:1000
					,localVsInternetTime = this.nowTime - Date.now()
					;
					
				if(this.endTime&&this.nowTime&&this.endTime>this.nowTime){
					clearTimeout(this.timer);
					// 确认服务器时间和当前时间的差值
					// setInterval消耗过大，当手机息屏时会自动停止，亮屏时开始自动运行，所以每次都要新取当前时间
					vm.nowTime+=intervalTime;
					this.timer=setInterval(function(){
						vm.nowTime = Date.now() + localVsInternetTime;
					}, intervalTime);
				}
			}
		},
		methods:{
			
		}
	}
</script>