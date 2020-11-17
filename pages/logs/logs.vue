<template>
<!--logs.wxml-->
<view class="container log-list">
  <block v-for="(log, index) in logs" :key="index">
    <text class="log-item">{{index + 1}}. {{log}}</text>
  </block>
</view>
</template>


<script>

global['__wxRoute'] = 'pages/logs/logs';
//logs.js
const util = require('../../utils/util.js')

Page({
  data: {
    logs: []
  },
  onLoad: function () {
    this.setData({
      logs: (wx.getStorageSync('logs') || []).map(log => {
        return util.formatTime(new Date(log))
      })
    })
  }
})

export default global['__wxComponents']['pages/logs/logs'];
</script>
<style>
.log-list {
  display: flex;
  flex-direction: column;
  padding: 40rpx;
}
.log-item {
  margin: 10rpx;
}

</style>