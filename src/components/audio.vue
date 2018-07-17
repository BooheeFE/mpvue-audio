<template>
  <div class="audio">
    <div :class="['cover-bg', play_status?'gray-bg':'normal-bg']">
      <img
        class="btn_pre"
        v-if="is_show_pre && !play_status"
        @click="onClickPre"
        src="../../static/images/ic_pre.png"
      >
        <img
          class="btn_next"
          v-if="is_show_next && !play_status"
          @click="onClickNext"
          src="../../static/images/ic_next.png"
        >
          <img
            class="btn_play"
            v-if="play_status"
            @click="onClickPlay"
            src="../../static/images/ic_play.png"
          >
            <img
              class="btn_play"
              v-if="!play_status"
              @click="onClickPause"
              src="../../static/images/ic_pause.png"
            >
              <div class="progress">
                <p class="text-left">{{progress}}</p>
                <slider
                  block-color="rgba(0,229,155,1)"
                  @change="sliderChange"
                  activeColor="rgba(0,229,155,0.9)"
                  backgroundColor="#ffffff"
                  :value="slide"
                  min="0"
                  :max="origin_length+1"
                  block-size="12"
                />
                <p class="text-right">{{audio_length}}</p>
              </div>
    </div>
    <img
      class="cover"
      :src="play_task ? play_task.content_cover_url:''"
    />
  </div>
</template>

<script>
export default {
  props: {
    task: {
      type: Object,
      default: null
    },
    task_array: {
      type: Array,
      default: () => {
        return []
      }
    }
  },

  data() {
    return {
      progress: '0:00',
      play_status: true,
      audio_length: '',
      audit_ctx: null,
      slide: 0,
      origin_length: 0,
      src: null,
      play_task: null,
      audio_index: 0,
      is_show_pre: false,
      is_show_next: false
    }
  },

  onLoad() {
    this.initAudio()
  },

  methods: {
    /**
     * 初始化播放器
     */
    initAudio() {
      this.play_task = this.getPlayTask()

      this.initNextPreStatus()

      console.log('initAudio', this.play_task)

      wx.stopBackgroundAudio()
      // 判断audio上下文
      if (!this.audit_ctx) {
        this.audit_ctx = wx.getBackgroundAudioManager()
      }

      this.audit_ctx.onPlay(() => {
        console.log('audit_ctx', 'onPlay')
        this.play_status = false
      })
      this.audit_ctx.onPause(() => {
        console.log('audit_ctx', 'onPause')
        this.play_status = true
      })
      this.audit_ctx.onTimeUpdate(() => {
        if (!this.origin_length) {
          this.audio_length = this.getTime(parseInt(this.audit_ctx.duration))
          if (this.audio_length === '00:00') {
            this.audio_length = ''
          }
          this.origin_length = parseInt(this.audit_ctx.duration) // 进度条
        }
        this.progress = this.getTime(parseInt(this.audit_ctx.currentTime))
        this.slide = parseInt(this.audit_ctx.currentTime)
      })
      this.audit_ctx.onEnded(() => {
        this.handleAudioEnded()
      })

      this.audit_ctx.onStop(() => {
        this.clearAudioStatus()
      })
    },
    /**
     * 处理音频结束的事件
     */
    handleAudioEnded() {
      const isSingle = this.task !== null
      if (isSingle) {
        this.clearAudioStatus()
      } else {
        // 判断是否是最后一首 如果是最后一首则 清空
        if (this.isLastIndexAudio()) {
          this.clearAudioStatus()
        } else {
          // 不是最后一首就到下一首播放
          this.playNextAudio()
        }
      }
    },
    /**
     * 播放下一首
     */
    playNextAudio() {
      this.audio_index = this.audio_index + 1
      this.play_task = this.task_array[this.audio_index]
      this.playAudio()
    },
    /**
     * 判断是否是最后一首
     */
    isLastIndexAudio() {
      return this.audio_index === this.task_array.length - 1
    },
    /**
     * 音频播放相关
     */
    sliderChange(event) {
      console.log(event)
      const e = event.mp
      let value =
        e.detail.value > this.origin_length
          ? this.origin_length - 1
          : e.detail.value
      this.audit_ctx.seek(value)
      this.slide = parseInt(value)
    },
    /**
     * 音频播放时间过滤器
     */
    getTime(num) {
      let a = num % 60 > 9 ? num % 60 : '0' + num % 60
      let b =
        Math.floor(num / 60) > 9
          ? Math.floor(num / 60)
          : '0' + Math.floor(num / 60)
      return b + ':' + a
    },
    /**
     * 清楚播放状态
     */
    clearAudioStatus() {
      this.src = ''
      this.slide = 0
      this.progress = '00:00'
      this.play_status = true
      this.audio_length = ''
      this.origin_length = 0
    },
    /**
     * 初始化媒体播放数据
     */
    getPlayTask() {
      let play_task
      if (this.task) {
        play_task = this.task
      } else {
        if (this.task_array && this.task_array.length > 0) {
          play_task = this.task_array[0]
        }
      }
      return play_task
    },
    /**
     * 播放
     */
    onClickPlay() {
      console.log('play')
      this.play_status = !this.play_status
      if (!this.src) {
        this.play_task = this.getPlayTask()
        this.playAudio()
      } else {
        this.audit_ctx.play()
      }
    },
    /**
     * 播放音频
     */
    playAudio() {
      const sys = wx.getSystemInfoSync().platform
      if (sys === 'ios') {
        wx.playBackgroundAudio({
          dataUrl: this.play_task.content_url,
          title: this.play_task.title,
          coverImgUrl: this.play_task.content_cover_url
        })
      } else {
        this.audit_ctx.src = this.play_task.content_url
        this.audit_ctx.title = this.play_task.title
        this.audit_ctx.coverImgUrl = this.play_task.content_cover_url
      }
      this.src = this.play_task.content_cover_url
    },
    /**
     * 停止
     */
    onClickPause() {
      this.audit_ctx.pause()
      this.play_status = !this.play_status
    },
    /**
     * 给外部提供的一个结束音频播放的方法
     */
    stop() {
      if (this.audit_ctx != null) {
        this.audit_ctx.stop()
      }
      this.clearAudioStatus()
    },
    /**
     * 初始化下一首和上一首的图标
     */
    initNextPreStatus() {
      if (this.task_array && this.task_array.length > 1) {
        this.is_show_next = true
        this.is_show_pre = false
      } else {
        this.is_show_next = false
        this.is_show_pre = false
      }
    },
    /**
     * 根据索引来显示隐藏 下一首上一首按钮
     */
    initNextPreByAudioIndex() {
      if (this.audio_index === 0) {
        this.is_show_pre = false
        // 判断长度是否大于1 如果是 则显示下一首
        if (this.task_array && this.task_array.length > 1) {
          this.is_show_next = true
        } else {
          this.is_show_next = false
        }
      } else if (this.audio_index === this.task_array.length - 1) {
        // 如果是最后一个
        this.is_show_next = false
        if (this.audio_index === 0) {
          this.is_show_pre = false
        } else {
          this.is_show_pre = true
        }
      } else {
        this.is_show_pre = true
        this.is_show_next = true
      }
    },
    /**
     * 点击上一首
     */
    onClickPre() {
      this.audio_index = this.audio_index - 1
      this.play_task = this.task_array[this.audio_index]
      this.clearAudioStatus()

      this.play_status = false

      this.playAudio()

      this.initNextPreByAudioIndex()
    },
    /**
     * 点击下一首
     */
    onClickNext() {
      this.audio_index = this.audio_index + 1
      this.play_task = this.task_array[this.audio_index]
      this.clearAudioStatus()

      this.play_status = false

      this.playAudio()

      this.initNextPreByAudioIndex()
    }
  },

  watch: {
    task() {
      console.log('watch', this.task)
      this.initAudio()
    },
    task_array() {
      this.initAudio()
    }
  }
}
</script>

<style lang="less" scoped>
.cover-bg {
  transition: 0.5s background-color;
  .progress {
    position: absolute;
    display: flex;
    align-items: center;
    font-size: 12px;
    justify-content: space-between;
    height: 34rpx;
    right: 40rpx;
    left: 40rpx;
    bottom: 24rpx;
    color: rgba(42, 163, 255, 0.9);
    slider {
      width: 504rpx;
      margin: 0;
    }
    .text-left {
      color: #ffffff;
      font-size: 24rpx;
      width: 64rpx;
    }
    .text-right {
      color: #ffffff;
      font-size: 24rpx;
      width: 64rpx;
    }
  }
}

.audio {
  .gray-bg {
    background-color: rgba(0, 0, 0, 0.3);
  }

  .normal-bg {
    background-color: rgba(0, 0, 0, 0);
  }
}

.audio {
  position: relative;
  width: 100%;
  height: 100%;
  .cover {
    width: 100%;
    height: 100%;
  }
  .cover-bg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    .btn_pre {
      position: absolute;
      top: 0;
      left: 20rpx;
      bottom: 0;
      margin: auto;
      z-index: 1;
      width: 92rpx;
      height: 92rpx;
    }
    .btn_next {
      position: absolute;
      top: 0;
      right: 20rpx;
      bottom: 0;
      margin: auto;
      z-index: 1;
      width: 92rpx;
      height: 92rpx;
    }
    .btn_play {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      margin: auto;
      z-index: 1;
      width: 92rpx;
      height: 92rpx;
    }
  }
}
</style>
