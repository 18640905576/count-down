# count-down
vue单文件组件构成的倒计时组件，可配置样式

可配置项
			nowTime:{type:Number,default:+new Date()},//当前时间
			endTime:{type:Number,default:0},//倒计时结束时间
			style:{type:String,default:''}, //样式，不填为黑色背景
			spaceType:{type:Number,default:1}, 
			//时间之间的连接方式，{1：符号【x天:xx:xx:xx.x】，2：中文文字【x天xx小时xx分xx秒x】，3：不填【x xx xx xx x】}
			//默认颜色跟着背景颜色走，如果需要改则在页面中用css改
			format:{type:String,default:'hh:mm:ss'}, //'dd hh:mm:ss.ms'分别对应天 小时 分 秒 十分之一秒，每一个都为可选的，默认显示时分秒
			pad:{type:Boolean,default:true} // 是否将时、分、秒格式化两位，空位用0填充，默认为填充
