<template>
<div class="fm-player">
  <div class="header">
    <span class="left">
      <div class="ball"></div>
      <div class="ball"></div>
      <div class="ball"></div>
    </span>
    <div class="mid">douban.fm</div>
    <img class="right" src='../assets/volume.png'>
  </div>
  <div class="nav">
      <div class="container">
        <div class="wrap">
          <div class="preStyle">{{channels[curIndex-1].name}}</div>
          <div class="curStyle">{{channels[curIndex].name}}</div>
          <div class="nextStyle">{{channels[curIndex+1].name}}</div>
        </div>
      </div>
      <div class="leftaside" @click='preStyle'><</div>
      <div class="rightaside" @click=nextStyle>></div>
  </div>
  <div class="main">
      <img :src="song.picture" class="cover">
      <div class="content">
      <h3>{{song.artist}}</h3>
      <div class="song">
      <p>{{song.title}}</p>
      <div class="time">-{{remainingTime}}</div>
      </div>
      <div class="progress-bar" @click='skip'>
            <div class="progress" v-bind:style='styleObject'></div>
        </div>
        <div class="icon">
          <img src="../assets/heart.gif">
          <img  class='play' src="../assets/next.png" @click='play' v-if='!playing'>
          <img  class='pause' src="../assets/pause.png" @click='pause' v-if='playing'>
          <img  class='next' src="../assets/next.gif" @click='getSong'>
        </div>
      </div>
  </div>
  <div class="lyric">{{changeCurLyric()}}</div>
</div>
</template>

<script>
export default {
  name: 'llj',
  data () {
    return {
      playing: false,
      song: {},
      channels: [{}, {}, {}],
      curIndex: 1,
      remainingTime: '00:00',
      curTime: '00:00',
      progressWidth: 0,
      fullLyric: '',
      curLyric: ''
    }
  },
  computed: {
    audio () {
      let audio = new Audio(this.song.url)
      audio.ontimeupdate = () => {
        let remainingSeconds = this.audio.duration - this.audio.currentTime
        let minutes = parseInt(remainingSeconds/60)
        let seconds = parseInt(remainingSeconds - 60*minutes)
        let curMinutes = parseInt((this.audio.currentTime)/60)
        let curSeconds = parseInt(this.audio.currentTime - 60*curMinutes)
        if(isNaN(minutes)){
          this.remainingTime = '00:00'
        }else{
          seconds = seconds>9 ? seconds : '0' + seconds
          minutes = minutes>9 ? minutes : '0' + minutes
          curSeconds = curSeconds>9 ? curSeconds : '0' + curSeconds
          curMinutes = curMinutes>9 ? curMinutes : '0' + curMinutes
          this.remainingTime = minutes + ':' + seconds
          this.curTime = curMinutes + ':' + curSeconds
          this.progressWidth = (this.audio.currentTime/this.audio.duration)*360
        }
        return
      }
      return audio
    },
    styleObject() {
      return {width: this.progressWidth + 'px'}
    }
  },
  methods: {
    preStyle () {
      this.curIndex--
    },
    nextStyle () {
      this.curIndex++
    },
    play () {
      this.playing = true
      this.audio.play()
    },
    pause () {
      this.playing = false
      this.audio.pause()
    },
    getSong () {
      new Promise((resolve,reject) => {
        let xhr = new XMLHttpRequest()
        xhr.open('get', 'http://api.jirengu.com/fm/getSong.php?channel=' + this.channels[this.curIndex].channel_id)
        xhr.onload = () => {
          this.pause()
          this.song = JSON.parse(xhr.responseText).song[0]
          resolve()
      }
      xhr.send()
      }).then(() => {
        let xhr = new XMLHttpRequest()
        xhr.open('get', 'http://jirenguapi.applinzi.com/fm/getLyric.php?&sid=' + this.song.sid)
        xhr.onload = () => {
          this.fullLyric = JSON.parse(xhr.responseText).lyric
          console.log(this.fullLyric)
        }
        xhr.send()
      })
    },
    skip(event) {
      this.progressWidth = event.offsetX
      this.audio.currentTime = (this.progressWidth/360)*this.audio.duration
    },
    changeCurLyric() {
      let str = '\\[' + this.curTime + '.\\d+\\].*'
      let reg1 = new RegExp(str)
      let reg2 = /[^\d\[\].:]+/
      if(reg1.test(this.fullLyric)){
        let lyric = this.fullLyric.match(reg1)[0]
        if(reg2.test(lyric)){
          this.curLyric = lyric.match(reg2)[0]
        }
      }
      return this.curLyric
    }
  },
  created () {
    new Promise((resolve, reject) => {
      let xhr = new XMLHttpRequest()
      xhr.open('get', 'http://api.jirengu.com/fm/getChannels.php')
      xhr.onload = () => {
        this.channels = JSON.parse(xhr.responseText).channels
        resolve()
      }
      xhr.send()
    }).then(() => {
      new Promise((resolve,reject) => {
        let xhr = new XMLHttpRequest()
        xhr.open('get', 'http://api.jirengu.com/fm/getSong.php?channel=' + this.channels[this.curIndex].channel_id)
        xhr.onload = () => {
          this.song = JSON.parse(xhr.responseText).song[0]
          resolve()
        }
        xhr.send()
      }).then(() => {
          let xhr = new XMLHttpRequest()
          xhr.open('get', 'http://jirenguapi.applinzi.com/fm/getLyric.php?&sid=' + this.song.sid)
          xhr.onload = () => {
            this.fullLyric = JSON.parse(xhr.responseText).lyric
            console.log(this.fullLyric)
        }
        xhr.send()
      })
    })
  }
}
</script>

<style>
    .fm-player{
      width: 680px
    }
  .header{
    border-radius: 10px 10px 0 0;
    height: 70px;
    background: rgb(5,165,160)
  }
  .header .left{
    display: inline-block;
    width: 240px;
    padding-left: 20px

  }
  .header .left .ball{
    width: 20px;
    height: 20px;
    border-radius: 10px;
    display: inline-block;
    margin-left: 10px;
    background: rgba(255,255,255,0.4);
    box-shadow: 0 0 1px 1px rgba(0,0,0,0.2) inset, 0 -2px 2px 2px rgb(11, 152, 156) inset, 0 1px 1px 1px rgba(255,255,255,0.2), 0 -1px 1px 0px rgba(0,0,0,0.2);
    margin-left: 10px
  }
  .header .mid{
    display: inline-block;
    font-size: 30px;
    color: white;
    width: 200px;
  }
  .header .right{
    margin-left: 140px;
    margin-top: 20px;
  }
  .nav{
    background: rgb(57,59,60);
    height: 65px;
  }
  .container,.leftaside,.rightaside{
    float: left
  }
  .wrap{
    width: 600px;
    margin: 0 40px;
  }
  .leftaside,.rightaside{
    width: 40px;
    height: 67px;
    font-size: 30px;
    line-height: 67px;
    text-align: center;
    color: rgb(123,127,124);
  }
  .leftaside{
    margin-left: -100%
  }
  .rightaside{
    margin-left: -40px;
  }
  .nav .wrap div{
    float: left;
    width: 200px;
    line-height: 65px;
    font-size: 35px;
    text-align: center;
    color: white;
  }
  .nav .wrap .preStyle,.nav .wrap .nextStyle{
    color: rgb(123,127,124);
    font-size: 30px
  }
  .main{
    padding-top: 30px;
    padding-left: 40px;
    height: 250px;
    background: rgb(232,237,234);
    border-radius: 0 0 10px 10px;
    box-sizing: border-box;
  }
  .main .cover{
    float: left;
    height: 170px
  }
  .content{
    float: left;
    width: 360px;
    height: 170px;
    margin-left: 80px;
  }
  .main h3{
    font-size: 20px;
    color: rgb(123,127,124);
    margin-top: 0;
    margin-bottom: 0
  }
  .main p,.main .time{
    float: left;
    font-size: 20px;
    margin-top: 0;
    margin-bottom: 0
  }
  .main .time{
    float: right;
  }
  .main .song:after{
    display: block;
    content: '';
    clear: both
  }
  .main .progress-bar{
    margin-top: 20px;
    height: 3px;
    background: rgb(206,217,211);
  }
  .main .progress-bar .progress{
    height: 100%;
    background: rgb(145,190,180);
  }
  .main .icon{
    margin-top: 35px
  }
  .main .icon img{
    float: left;
    width: 46px;
    height: 40px;
  }
  .main .icon .next{
    float: right;
  }
  .main .icon .play,.pause{
    margin-left: 110px;
  }
  .lyric{
    text-align: center;
    margin: 0 auto;
    color: rgb(145,190,180)
  }
</style>
