<template>
	<!--pages/list/list.wxml-->
	<scroll-view class="info-panel">
		<view class="head-panel">
			<image class="avatar" :src="info.avatar_url"></image>
			<view class="action-panel">
				<view class="button red-button button-s" @tap="onCall">打电话</view>
				<view class="button red-button button-s" @tap="onOpenDY">跳主页</view>
			</view>
		</view>
		<view class="name">{{ info.nickname }}</view>
		<view class="info-s" @tap="clipId" v-if="info.short_id">抖音ID(点击复制):{{ info.short_id }}</view>
		<view class="info-s" @tap="clipCall" v-if="info.main_phone">Tel(点击复制):{{ info.main_phone }}</view>
		<view class="info-s">{{ info.up_time }}</view>
		<view class="line"></view>
		<view class="text">{{ info.signature }}</view>
		<view class="line"></view>
		<view class>{{ info.aweme_count }}作品 {{ info.total_favorited }}赞 平均{{ info.total_favorited / info.aweme_count }}</view>
		<view class="line"></view>
		<view class="button-panel">
			<view class="button button-s red-button" @tap="updateType" data-type="-9" data-str="不是商户">不是商户</view>
			<view class="button button-s red-button" @tap="updateType" data-type="-7" data-str="电话错误">电话错误</view>
			<view class="button button-s red-button" @tap="updateType" data-type="-8" data-str="没有意向">没有意向</view>
			<view class="button button-s red-button" @tap="updateType" data-type="-6" data-str="同行正在做">同行正在做</view>
			<view class="button button-s red-button" @tap="updateType" data-type="3" data-str="稍候联系">稍候联系</view>
			<view class="button button-s red-button" @tap="updateType" data-type="5" data-str="考虑考虑">考虑考虑</view>
			<view class="button button-s red-button" @tap="updateType" data-type="6" data-str="意向强烈">意向强烈</view>
			<view class="button button-s red-button" @tap="updateType" data-type="9" data-str="已申请微信">已申请微信</view>
			<view v-if="info" class="button button-s  red-button" @tap="next">下一个</view>
		</view>
	</scroll-view>
</template>

<script>
// 引入文件
import uniCopy from '@/js_sdk/xb-copy/uni-copy.js';

export default {
	data() {
		return {
			userName: '',
			branch:'',
			info: {},
			crmType:null
		};
	},
	onLoad(options) {
		if (options.userName) {
			uni.setStorage({
				data: options.userName,
				key: 'userName'
			});
			this.$data.userName = options.userName;
		} else {
			var userName = uni.getStorageSync('userName');
			if (userName) {
				this.$data.userName = userName;
			}
		}
		console.log(this.$data.userName);
		if (options.branch) {
			uni.setStorage({
				data: options.branch,
				key: 'branch'
			});
			this.$data.branch = options.branch;
		} else {
			var branch = uni.getStorageSync('branch');
			if (branch) {
				this.$data.branch = branch;
			}
		}
		console.log(this.$data.branch);
		this.getInfo();
	},
	onReady() {},
	methods: {
		//打电话
		onCall: function() {
			var that = this;
			uni.request({
				url: this.websiteUrl,
				data: {
					method: 'onCall',
					id: this.$data.info.id,
					user: this.$data.userName
				},
				success() {
					uni.makePhoneCall({
						phoneNumber: that.$data.info.main_phone
					});
				}
			});
		},
		//打开抖音
		onOpenDY: function() {
			var that = this;
			uni.request({
				url: this.websiteUrl,
				data: {
					method: 'onOpenDY',
					id: this.$data.info.id,
					user: this.$data.userName
				},
				success() {
					window.location = 'snssdk1128://user/profile/' + that.$data.info.uid;
				}
			});
		},
		copyTest: function() {
			this.forceCopy("打发打发士大夫")
		},
		
		//复制手机号
		clipCall: function() {
			//请求
			uni.request({
				url: this.websiteUrl,
				data: {
					method: 'onClipDY',
					id: this.$data.info.id,
					user: this.$data.userName
				}
			});
			//复制到剪贴板
			let content = this.$data.info.main_phone;
			content = typeof content === 'string' ? content : content.toString(); // 复制内容，必须字符串，数字需要转换为字符串
			if (uni.getSystemInfoSync().platform == 'ios') {
				this.forceCopy(this.$data.info.main_phone);
			} else {
				uniCopy(content)
			}
			uni.showToast({
				title: '复制成功'
			});
		},

		//复制抖音号
		clipId: function() {
			uni.request({
				url: this.websiteUrl,
				data: {
					method: 'onClipTel',
					id: this.$data.info.id,
					user: this.$data.userName
				}
			});
			let content = this.$data.info.short_id;
			content = typeof content === 'string' ? content : content.toString(); // 复制内容，必须字符串，数字需要转换为字符串
			if (uni.getSystemInfoSync().platform == 'ios') {
				this.forceCopy(this.$data.info.short_id);
			} else {
				uniCopy(content);
			}
			uni.showToast({
				title: '复制成功'
			});
		},
		next: function() {
			if(this.$data.crmType||this.$data.info == null){
				this.getInfo();
			}else{
				uni.showToast({
					title:"请标注客户意向,并对标注结果负责",
					icon:'none'
				});
			}
			
		},
		getInfo: function() {
			var that = this;

			uni.request({
				url: this.websiteUrl,
				data: {
					method: 'getInfo',
					user: this.$data.userName,
					branch:this.$data.branch
				},
				success: function(res) {
					console.log(res);
					var data = res.data;
					that.$data.info = data.data;
					that.$data.crmType = null;
				}
			});
		},
		updateType: function(e) {
			var type = e.currentTarget.dataset.type;
			var typeStr = e.currentTarget.dataset.str;
			this.$data.crmType = type;
			uni.request({
				url: this.websiteUrl,
				data: {
					method: 'update_crm',
					id: this.$data.info.id,
					type: type,
					type_str: typeStr,
					user: this.$data.userName
				},
				success: function(res) {
					uni.showToast({
						title: '已标记'
					});
				}
			});
		},
		forceCopy: function(text) {

			// 数字没有 .length 不能执行selectText 需要转化成字符串
				const textString = text.toString();
				let input = document.querySelector('#copy-input');
				if (!input) {
				  input = document.createElement('input');
				  input.id = "copy-input";
				  input.readOnly = "readOnly";        // 防止ios聚焦触发键盘事件
				  input.style.position = "absolute";
				  input.style.left = "-1000px";
				  input.style.zIndex = "-1000";
				  document.body.appendChild(input)
				}
				input.value = textString;
				// ios必须先选中文字且不支持 input.select();
				this.selectText(input, 0, textString.length);
				console.log(document.execCommand('copy'), 'execCommand');
				if (document.execCommand('copy')) {
				  document.execCommand('copy');
				}
				input.blur();
				uni.showToast({
					title: 'ios:复制成功'
				});
			
		},
		selectText: function(textbox, startIndex, stopIndex) {
		      if (textbox.createTextRange) {//ie
		        const range = textbox.createTextRange();
		        range.collapse(true);
		        range.moveStart('character', startIndex);//起始光标
		        range.moveEnd('character', stopIndex - startIndex);//结束光标
		        range.select();//不兼容苹果
		      } else {//firefox/chrome
		        textbox.setSelectionRange(startIndex, stopIndex);
		        textbox.focus();
		      }
		}
	}
};
</script>
<style>
/* pages/list/list.wxss */
page {
	color: #fff;
	background-color: #161922;
	padding-left: 5%;
}

.red-button {
	background-color: #ff2d54;
}

.head-panel {
	width: 90%;
	padding: 60rpx 0 30rpx 0;
	display: inline-flex;
	flex-direction: row;
	align-items: center;
	justify-content: space-between;
}

.avatar {
	width: 200rpx;
	height: 200rpx;
	border-radius: 100%;
}

.button {
	height: 80rpx;
	width: 260rpx;
	border-radius: 5rpx;
	line-height: 80rpx;
	text-align: center;
}

.name {
	font-size: 1.4em;
	padding: 30rpx 0;
}

.info-s {
	padding-top: 10rpx;
}

.line {
	margin: 50rpx 0;
	height: 1px;
	width: 90%;
	color: rgba(255, 255, 255, 0.7);
	border-bottom: rgba(255, 255, 255, 0.7) 1px solid;
}

.text {
	width: 90%;
	min-height: 200rpx;
}

.button-panel {
	width: 90%;
	display: flex;
	flex-direction: row;
	align-items: center;
	justify-content: space-between;
	flex-wrap: wrap;
}

.button-s {
	margin-bottom: 20rpx;
	width: 200rpx;
}
.action-panel {
	display: flex;
	flex-direction: column;
	align-items: center;
}
</style>
