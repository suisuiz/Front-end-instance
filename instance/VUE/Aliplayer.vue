<!--
 * @Descripttion: 
 * @Author: SUI
 * @Date: 2021-12-20 17:16:52
 * @LastEditors: SUI
 * @LastEditTime: 2021-12-20 17:16:53
 * @FilePath: \advertising-screen\src\components\Aliplayer.vue
-->
<template>
  <div class="hello">
    <div class="prism-player" :id="playerId" :style="playStyle"></div>
  </div>
</template>

<script>
export default {
  name: 'Aliplayer',
  props: {
    playStyle: {
      type: String,
      default: ''
    },
    aliplayerSdkPath: {
      // Aliplayer 代码的路径
      type: String,
      default: '//g.alicdn.com/de/prismplayer/2.6.0/aliplayer-min.js'
    },
    autoplay: {
      type: Boolean,
      default: true
    },
    isLive: {
      type: Boolean,
      default: false
    },
    playsinline: {
      // H5是否内置播放，有的Android浏览器不起作用。
      type: Boolean,
      default: false
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '100%'
    },
    controlBarVisibility: {
      // 控制面板的实现，默认为‘hover’， 可选的值为：‘click’、‘hover’、‘always’。
      type: String,
      default: 'hover'
    },
    useH5Prism: {
      type: Boolean,
      default: false
    },
    useFlashPrism: {
      type: Boolean,
      default: false
    },
    vid: {
      type: String,
      default: ''
    },
    playauth: {
      type: String,
      default: ''
    },
    source: {
      type: String,
      default: ''
      //这个换成你的视频地址，因为我做的是视频监控，在这里放的rtmp推流地址
    },
    cover: {
      type: String,
      default: ''
    },
    format: {
      type: String,
      default: 'm3u8'
    },
    x5_video_position: {
      type: String,
      default: 'top'
    },
    x5_type: {
      type: String,
      default: 'h5'
    },
    x5_fullscreen: {
      type: Boolean,
      default: false
    },
    x5_orientation: {
      type: Number,
      default: 2
    },
    autoPlayDelay: {
      type: Number,
      default: 0
    },
    autoPlayDelayDisplayText: {
      type: String
    }
  },
  data() {
    return {
      playerId: 'aliplayer_' + Math.random().toString(36).substr(2),
      scriptTagStatus: 0,
      isReload: false,
      instance: null
    }
  },
  created() {
    if (window.Aliplayer !== undefined) {
      // 如果全局对象存在，说明编辑器代码已经初始化完成，直接加载编辑器
      this.scriptTagStatus = 2
      this.initAliplayer()
    } else {
      // 如果全局对象不存在，说明编辑器代码还没有加载完成，需要加载编辑器代码
      this.insertScriptTag()
    }
  },
  mounted() {
    if (window.Aliplayer !== undefined) {
      // 如果全局对象存在，说明编辑器代码已经初始化完成，直接加载编辑器
      this.scriptTagStatus = 2
      this.initAliplayer()
    } else {
      // 如果全局对象不存在，说明编辑器代码还没有加载完成，需要加载编辑器代码
      this.insertScriptTag()
    }
  },
  methods: {
    insertScriptTag() {
      const _this = this
      let playerScriptTag = document.getElementById('playerScriptTag')
      // 如果这个tag不存在，则生成相关代码tag以加载代码
      if (playerScriptTag === null) {
        playerScriptTag = document.createElement('script')
        playerScriptTag.type = 'text/javascript'
        playerScriptTag.src = this.aliplayerSdkPath
        playerScriptTag.id = 'playerScriptTag'
        let s = document.getElementsByTagName('head')[0]
        s.appendChild(playerScriptTag)
      }
      if (playerScriptTag.loaded) {
        _this.scriptTagStatus++
      } else {
        playerScriptTag.addEventListener('load', () => {
          _this.scriptTagStatus++
          playerScriptTag.loaded = true
          _this.initAliplayer()
        })
      }
      _this.initAliplayer()
    },
    initAliplayer() {
      const _this = this
      // scriptTagStatus 为 2 的时候，说明两个必需引入的 js 文件都已经被引入，且加载完成
      if (_this.scriptTagStatus === 2 && (_this.instance === null || _this.reloadPlayer)) {
        _this.instance && _this.instance.dispose()
        // Vue 异步执行 DOM 更新，这样一来代码执行到这里的时候可能 template 里面的 script 标签还没真正创建
        // 所以，我们只能在 nextTick 里面初始化 Aliplayer
        _this.$nextTick(() => {
          _this.instance = window.Aliplayer(
            {
              id: _this.playerId,
              autoplay: _this.autoplay,
              isLive: _this.isLive,
              playsinline: _this.playsinline,
              format: _this.format,
              width: _this.width,
              height: _this.height,
              controlBarVisibility: _this.controlBarVisibility,
              useH5Prism: _this.useH5Prism,
              useFlashPrism: _this.useFlashPrism,
              vid: _this.vid,
              playauth: _this.playauth,
              source: _this.source,
              cover: _this.cover,
              x5_video_position: _this.x5_video_position,
              x5_type: _this.x5_type,
              x5_fullscreen: _this.x5_fullscreen,
              x5_orientation: _this.x5_orientation,
              autoPlayDelay: _this.autoPlayDelay,
              autoPlayDelayDisplayText: _this.autoPlayDelayDisplayText
            },
            (player) => {
              //先静音然后播放
              player.mute()
              player.play()
            }
          )
          // 绑定事件，当 AliPlayer 初始化完成后，将编辑器实例通过自定义的 ready 事件交出去
          _this.instance.on('ready', () => {
            console.log('ready')
            this.$emit('ready', _this.instance)
          })
          _this.instance.on('play', () => {
            console.log('play')
            this.$emit('play', _this.instance)
          })
          _this.instance.on('pause', () => {
            console.log('pause')
            this.$emit('pause', _this.instance)
          })
          _this.instance.on('ended', () => {
            this.$emit('ended', _this.instance)
          })
          _this.instance.on('liveStreamStop', () => {
            this.$emit('liveStreamStop', _this.instance)
          })
          _this.instance.on('m3u8Retry', () => {
            this.$emit('m3u8Retry', _this.instance)
          })
          _this.instance.on('hideBar', () => {
            this.$emit('hideBar', _this.instance)
          })
          _this.instance.on('waiting', () => {
            this.$emit('waiting', _this.instance)
          })
          _this.instance.on('snapshoted', () => {
            this.$emit('snapshoted', _this.instance)
          })

          _this.instance.on('timeupdate', () => {
            _this.$emit('timeupdate', _this.instance)
          })
          _this.instance.on('requestFullScreen', () => {
            _this.$emit('requestFullScreen', _this.instance)
          })
          _this.instance.on('cancelFullScreen', () => {
            _this.$emit('cancelFullScreen', _this.instance)
          })
          _this.instance.on('error', () => {
            _this.$emit('error', _this.instance)
          })
          _this.instance.on('startSeek', () => {
            _this.$emit('startSeek', _this.instance)
          })
          _this.instance.on('completeSeek', () => {
            _this.$emit('completeSeek', _this.instance)
          })
        })
      }
    },
    /**
     * 播放视频
     */
    play: function () {
      this.instance.play()
    },
    /**
     * 暂停视频
     */
    pause: function () {
      this.instance.pause()
    },
    /**
     * 重播视频
     */
    replay: function () {
      this.instance.replay()
    },
    /**
     * 跳转到某个时刻进行播放
     * @argument time 的单位为秒
     */
    seek: function (time) {
      this.instance.seek(time)
    },
    /**
     * 获取当前时间 单位秒
     */
    getCurrentTime: function () {
      return this.instance.getCurrentTime()
    },
    /**
     *获取视频总时长，返回的单位为秒
     * @returns 返回的单位为秒
     */
    getDuration: function () {
      return this.instance.getDuration()
    },
    /**
       获取当前的音量，返回值为0-1的实数ios和部分android会失效
      */
    getVolume: function () {
      return this.instance.getVolume()
    },
    /**
     设置音量，vol为0-1的实数，ios和部分android会失效
    */
    setVolume: function (vol) {
      this.instance.setVolume(vol)
    },
    /**
     *直接播放视频url，time为可选值（单位秒）目前只支持同种格式（mp4/flv/m3u8）之间切换暂不支持直播rtmp流切换
     *@argument url 视频地址
     *@argument time 跳转到多少秒
     */
    loadByUrl: function (url, time) {
      this.instance.loadByUrl(url, time)
    },
    /**
     * 设置播放速度
     *@argument speed 速度
     */
    setSpeed: function (speed) {
      this.instance.setSpeed(speed)
    },
    /**
     * 设置播放器大小w,h可分别为400px像素或60%百分比chrome浏览器下flash播放器分别不能小于397x297
     *@argument w 播放器宽度
     *@argument h 播放器高度
     */
    setPlayerSize: function (w, h) {
      this.instance.setPlayerSize(w, h)
    },
    /**
     * 目前只支持HTML5界面上的重载功能,暂不支持直播rtmp流切换m3u8）之间切换,暂不支持直播rtmp流切换
     *@argument vid 视频id
     *@argument playauth 播放凭证
     */
    reloaduserPlayInfoAndVidRequestMts: function (vid, playauth) {
      this.instance.reloaduserPlayInfoAndVidRequestMts(vid, playauth)
    },
    reloadPlayer: function () {
      this.isReload = true
      this.initAliplayer()
      this.isReload = false
    }
  }
}
</script>

<style lang="postcss" scoped>
@import 'https://g.alicdn.com/de/prismplayer/2.7.2/skins/default/aliplayer-min.css';
</style>
