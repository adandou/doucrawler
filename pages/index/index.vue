<template>
<!--index.wxml-->
<view class="container">
  <view class="userinfo">
    <button v-if="!userName"> 请登陆系统 </button>
    <block v-else>
      <image @tap="bindViewTap" class="userinfo-avatar" :src="avatarUrl" mode="cover"></image>
      <text class="userinfo-nickname">{{userName}},欢迎</text>
    </block>
  </view>
  <view v-if="userName" class="usermotto" @tap="onMotto">
    <text class="user-motto">{{motto}}</text>
  </view>
</view>
</template>


<script>
	
   export default {
        data() {
            return {
                userName: '',
				motto:"进入CRM终端",
				avatarUrl:'http://dycrm.homepsf.com/logo.jpg'
            }
        },
        onLoad(options) {
			if (options.userName) {
				uni.setStorage({
				  data: options.userName,
				  key: 'userName',
				})
				this.$data.userName = options.userName
			};
			if (options.branch) {
				uni.setStorage({
				  data: options.branch,
				  key: 'branch',
				})
			}
        },
        onShow() {
            console.log('页面显示')
        },
        onReady(){
            console.log('页面初次显示')
        },
        onHide() {
            console.log('页面隐藏')
        },
        onUnload() {
            console.log('页面卸载')
        },
        onBackPress(){
            console.log('页面返回...')
        },
        onShareAppMessage() {
            console.log('分享!')
        },
        onReachBottom() {
            console.log('下拉加载...')
        },
        onPageScroll(){
            console.log('页面滚动...')
        },
        onPullDownRefresh() {
            console.log('上拉刷新...')
            uni.stopPullDownRefresh();
        },

        // #ifdef APP-PLUS
        onNavigationBarButtonTap(){

        },
        // #endif

		methods: {
		  onMotto: function(){
			uni.navigateTo({
			  url: '../list/list?userName='+this.$data.userName
			})
		  },
	}

}
</script>
<style>
/**index.wxss**/
.userinfo {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.userinfo-avatar {
  width: 128rpx;
  height: 128rpx;
  margin: 20rpx;
  border-radius: 50%;
}

.userinfo-nickname {
  color: #aaa;
}

.usermotto {
  margin-top: 200px;
  padding: 30px;
}
</style>