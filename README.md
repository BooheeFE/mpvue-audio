# mpvue-audio

mpvue 微信小程序 音频播放组件

## 安装

```
npm install mpvue-audio --save
```

## 可用属性

|参数|说明|类型|默认值|
|---|---|---|---|
|task|单音频|Object|null|
|task_array|多音频|Array[Object]|[]|


### task说明

|参数|说明|类型|
|---|---|---|
|content_url|音频地址|String|
|title|标题|String|''|
|coverImgUrl|封面地址|String|

## 使用

```html
<template>
  <div class="audio-container">
    <my-audio :task="task" />
  </div>
</template>
```

```js
import MyAudio from 'mpvue-audio'

export default {
  data() {
    return {
      task: {}
    }
  },

  components: {
    MyAudio
  },

  methods: {
    //...省略

    initTask() {
      this.task.content_url = 'http://example.mp3'
      this.task.content_cover_url = 'http://example.png'
      this.task.title = 'example'
    }
  },

  created() {
    this.initTask()
  }
}
</script>
```


