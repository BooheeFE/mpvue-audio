<template>
  <div class="container" @click="clickHandle('test click', $event)">
    <div class="userinfo" @click="bindViewTap">
      <img class="userinfo-avatar" v-if="userInfo.avatarUrl" :src="userInfo.avatarUrl"
        background-size="cover" />
      <div class="userinfo-nickname">
        <card :text="userInfo.nickName"></card>
      </div>
    </div>
    <div class="audio-container">
      <my-audio :task="task" />
    </div>
  </div>
</template>

<script>
import card from '@/components/card'
import MyAudio from 'mpvue-audio'

export default {
  data() {
    return {
      motto: 'Hello World',
      userInfo: {},
      task: {}
    }
  },

  components: {
    card,
    MyAudio
  },

  methods: {
    bindViewTap() {
      const url = '../logs/main'
      wx.navigateTo({ url })
    },
    getUserInfo() {
      // 调用登录接口
      wx.login({
        success: () => {
          wx.getUserInfo({
            success: res => {
              this.userInfo = res.userInfo
            }
          })
        }
      })
    },
    clickHandle(msg, ev) {
      console.log('clickHandle:', msg, ev)
    },
    initTask() {
      this.task.content_url = 'https://raw.githubusercontent.com/Baifann/mpvue-audio/master/assets/audio/wind.mp3'
      this.task.content_cover_url = 'https://raw.githubusercontent.com/Baifann/mpvue-audio/master/assets/img/wind.jpg'
      this.task.title = '起风了'
    }
  },

  created() {
    // 调用应用实例的方法获取全局数据
    this.getUserInfo()
    this.initTask()
  }
}
</script>

<style scoped>
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
  margin-top: 150px;
}

.form-control {
  display: block;
  padding: 0 12px;
  margin-bottom: 5px;
  border: 1px solid #ccc;
}

.audio-container {
  width: 750rpx;
  height: 422rpx;
}
</style>
