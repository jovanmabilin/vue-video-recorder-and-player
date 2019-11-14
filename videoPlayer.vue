<template>
    <div>
          <div class="video-recorder-container">
            <b-row class="mb-0 mr-0 ml-0 mt-3 details-container">
              <b-col></b-col>
              <!-- <b-col class="text-white text-center">DETAILS</b-col> -->
              <b-col class="text-right"></b-col>
            </b-row>
            <div class="video-container">
              <video ref="videoPlayer" :src="videoSrc" @timeupdate="videoTimeUpdate"></video>
            </div>
            <b-row class="m-0 main-controls-container">
                <b-col></b-col>
                <!-- <b-col class=""><i class="fa fa-play text-white"></i></b-col> -->
                <b-col></b-col>
            </b-row>
            <b-row class="m-0 controls-container">
              <b-col cols="12" class="m-0 p-0">
                <!-- <input type="range" min="0" max="100" value="0" class="w-100 video-progress-bar" step="1" @change="seekTimePosition" v-model="sliderValue"> -->
                <div class="video-buffer-container" @click="seekTimePosition">
                    <div class="video-buffer-item circle" :style="circlePosition"></div>
                    <!-- <div class="video-buffer-item buffer-to-be-played"></div> -->
                    <div class="video-buffer-item buffer-played" :style="updateProgressBar"></div>
                </div>
                <div class="video-control-container bg-dark text-white">
                  <b-row>
                      <b-col class="text-left">
                          <span :class="['i fa', `fa-${ currentState }`]" @click="videoPlayPause"></span>
                      </b-col>
                      <b-col class="text-center">
                          <span class="video-time-tracker">{{ timeTracker(videoCurrentTime) }} / {{ timeTracker( videoDuration ) }}</span>
                      </b-col>
                      <b-col class="text-right">
                          <span class="i fa fa-expand" @click="toggleFullScreen"></span>
                      </b-col>
                  </b-row>
                </div>
                </b-col>
              </b-row>
          </div>
        <b-button @click="videoPlayPause" variant="primary">Play</b-button>
        <b-button @click="videoPause" variant="warning">Pause</b-button>
        <b-button @click="videoStop" variant="danger">Stop</b-button>


        <div class="mt-5">

        </div>
    </div>
</template>

<script>
const floor = Math.floor;
const zeroPrepend = int => (int > 9 ? `${int}` : `0${int}`);
const defaultTime = time => time || "00";
const returnValue = value => (value > 0 ? zeroPrepend(value) : "");
const setTimeFormat = (hour, min, sec, defaultFormat) =>
  hour || min || sec
    ? defaultTime(hour) + ":" + defaultTime(min) + ":" + defaultTime(sec)
    : defaultFormat;
const minute = time => floor((time % 3600) / 60);
const second = time => floor((time % 3600) % 60);
const hour = time => floor(time / 3600);


    export default {
        data() {
            return {
                videoSrc: require('../../assets/recording_video.mp4'),
                videoPlayer: null,
                videoDuration: 0,
                videoCurrentTime: 0,
                currentState: 'play',
                videoProgress: null,
                videoProgressPosition: 0,
                sliderValue: 0,
            }
        },
        computed: {
          updateProgressBar() {
            return {
              'width' : this.videoProgressPosition * 100 + '%' 
            }
          },
          circlePosition() {
            return {
              'left' : ( ( this.videoProgressPosition * 100 ) > 97  ) ? '97%' : this.videoProgressPosition * 100 + '%' 
            }
          }
        },
        methods: {
            timeFormatter(time) {
              let defaultFormat = "00:00:00";
              let secondFormat = returnValue(second(time));
              let minuteFormat = returnValue(minute(time));
              let hourFormat = returnValue(hour(time));

              return setTimeFormat(
                hourFormat,
                minuteFormat,
                secondFormat,
                defaultFormat
              );
            },
            timeTracker(leftTime, rightTime) {
              return this.timeFormatter(leftTime);
            },
            videoPlayPause() {
              if ( this.videoPlayer.paused ) {
                this.videoPlayer.play();
                this.currentState = 'pause';
              } else {
                this.videoPlayer.pause();
                this.currentState = 'play';
              }

              if ( this.videoPlayer.ended ) {
                this.currentState = 'play'
              }
            },
            videoPause() {
                this.videoPlayer.pause();
            },
            videoStop() {
                this.videoPlayer.load();
            },
            videoTimeUpdate(e) {
              this.videoDuration = this.videoPlayer.duration;
              this.videoCurrentTime = this.videoPlayer.currentTime;
              this.videoProgressPosition = this.videoCurrentTime / this.videoDuration;
              this.sliderValue = this.videoCurrentTime * ( 100 / this.videoDuration );
            },
            toggleFullScreen() {
              if ( this.videoPlayer.requestFullScreen ) {
                this.videoPlayer.requestFullScreen();
              } else if ( this.videoPlayer.webkitRequestFullScreen ) {
                this.videoPlayer.webkitRequestFullScreen();
              } else if ( this.videoPlayer.mozRequestFullScreen ) {
                this.videoPlayer.mozRequestFullScreen();
              }
            },
            seekTimePosition(e) {
              let percent = e.offsetX / e.offsetWidth;
              // this.videoCurrentTime = percent * this.videoDuration;
              // this.videoProgressPosition = percent * 100;
              console.log(e);

              let seekTo = this.videoDuration * ( this.sliderValue / 100 );
              this.videoPlayer.currentTime = seekTo;
            },
        },
        mounted() {
            this.videoPlayer = this.$refs.videoPlayer;
            this.videoDuration = this.videoPlayer.duration;
        },
    }
</script>

<style lang="scss" scoped>
.b-link-close {
  background-color:rgba(0,0,0,0.3);
  padding: 0px 5px 0px 5px;
}
.b-link-close:hover {
  opacity: 0.8;
}
.back-container {
  font-size: 12px;
  background-color: rgba(0,0,0,0.3);
  padding: 5px 10px 5px 10px;
  cursor: pointer !important;
  transition: 0.3s ease-out;
  border-radius: 50px;
}
.back-container:hover {
  text-decoration: underline;
  opacity: 0.8;
  transition: 0.3s ease-in;
}
.time-container {
  background-color: rgba(0, 0, 0, 0.3);
  padding: 5px 5px 5px 5px;
  border-radius: 50px;
  font-size: 12px;
  margin: 0 auto;
  width: 100px;
}

.circle-container {
  background-color: rgba(0, 0, 0, 0.3);
  width: 3rem;
  height: 3rem;
  border-radius: 50%;
  position: relative;
  margin: 0 auto;
}

.circle-container .circle-item {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: (2rem / 2);
  height: (2rem / 2);
  border-radius: 50%;
  background-color: #ff0000;
  transition: 0.3s ease-out;
}

.circle-item {
  cursor: pointer;
}

.square-item {
  cursor: pointer;
}

.circle-container .square-item {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #ff0000;
  width: (2rem / 2);
  height: (2rem / 2);
  border-radius: 5px;
  transition: 0.3s ease-out;
}

.circle-item:hover {
  width: (3rem / 2);
  height: (3rem / 2);
  transition: 0.3s ease-in;
}
.square-item:hover {
  width: (3rem / 2);
  height: (3rem / 2);
  transition: 0.3s ease-in;
}

.video-recorder-container {
  position: relative;
}

.main-controls-container {
    margin: 0;
    background: yellow;
    position: absolute;
    top: 50%;
    left: 50%;
    margin-right: -50%;
    transform: translate(-50%, -50%);
    z-index: 50;
}
.controls-container {
  position: absolute;
  width: 100%;
  bottom: 0;
  background: red;
  z-index: 50;
}

.details-container {
  position: absolute;
  width: 100%;
  top: 0;
  z-index: 50;
  background: green;
}

.record-container {
  animation: reduce-opacity 1s infinite;
}

video {
  width: 100% !important;
}

@keyframes shadow-pulse {
  0% {
    box-shadow: 0 0 0 0px rgba(255, 0, 0, 0.2);
    border-radius: 100%;
  }
  100% {
    box-shadow: 0 0 0 8px rgba(255, 0, 0, 0);
    border-radius: 100%;
  }
}
@keyframes reduce-opacity {
      0% { -webkit-filter:    opacity(1);}
      50% { -webkit-filter:   opacity(.5);}
      100% { -webkit-filter:  opacity(1);}
}

.video-control-container {
    padding: 10px 15px 10px 15px;
    font-size: 12px;
}

.video-buffer-container {
    background-color: rgba(107, 108, 109,1);
    width: 100%;
    height: 5px;
    position: relative;
    transition: 1s ease-in-out;
}

.video-buffer-item.circle {
    position: absolute;
    top: -5px;
    background-color: #ffffff;
    width: 15px;
    height: 15px;
    border-radius: 100%;
    box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.4);
    z-index: 10;
    transition: 1s ease-in-out;
}

.video-buffer-item.buffer-to-be-played {
    position: absolute;
    width: 80%;
    height: 5px;
    background-color: #8f9091;
    z-index: 8;
    transition: 1s ease-in-out;
}

.video-buffer-item.buffer-played {
    position: absolute;
    height: 5px;
    background-color: #ffffff;
    z-index: 9;
    transition: 1s ease-in-out;
}

input[type=range] {
  -webkit-appearance: none;
  width: 100%;
  margin: 4.85px 0;
}
input[type=range]:focus {
  outline: none;
}
input[type=range]::-webkit-slider-runnable-track {
  width: 100%;
  height: 6.3px;
  cursor: pointer;
  box-shadow: 0px 0px 0px #92fff7, 0px 0px 0px #acfff9;
  background: #6b6c6d;
  border: 0px solid #cb85cd;
}
input[type=range]::-webkit-slider-thumb {
  box-shadow: 0px 0px 2.6px #000000, 0px 0px 0px #0d0d0d;
  border: 0px solid #000000;
  height: 16px;
  width: 16px;
  background: #ffffff;
  cursor: pointer;
  -webkit-appearance: none;
  margin-top: -4.85px;
}
input[type=range]:focus::-webkit-slider-runnable-track {
  background: #78797a;
}
input[type=range]::-moz-range-track {
  width: 100%;
  height: 6.3px;
  cursor: pointer;
  box-shadow: 0px 0px 0px #92fff7, 0px 0px 0px #acfff9;
  background: #6b6c6d;
}
input[type=range]::-moz-range-thumb {
  box-shadow: 0px 0px 2.6px #000000, 0px 0px 0px #0d0d0d;
  border: 0px solid #000000;
  height: 16px;
  width: 16px;
  background: #ffffff;
  cursor: pointer;
}
input[type=range]::-ms-track {
  width: 100%;
  height: 6.3px;
  cursor: pointer;
  background: transparent;
  border-color: transparent;
  color: transparent;
}
input[type=range]::-ms-fill-lower {
  background: #5e5f60;
  border: 0px solid #cb85cd;
  box-shadow: 0px 0px 0px #92fff7, 0px 0px 0px #acfff9;
}
input[type=range]::-ms-fill-upper {
  background: #6b6c6d;
  border: 0px solid #cb85cd;
  box-shadow: 0px 0px 0px #92fff7, 0px 0px 0px #acfff9;
}
input[type=range]::-ms-thumb {
  box-shadow: 0px 0px 2.6px #000000, 0px 0px 0px #0d0d0d;
  border: 0px solid #000000;
  height: 16px;
  width: 16px;
  background: #ffffff;
  cursor: pointer;
  height: 6.3px;
}
input[type=range]:focus::-ms-fill-lower {
  background: #6b6c6d;
}
input[type=range]:focus::-ms-fill-upper {
  background: #78797a;
}
input[type=range] {
  -webkit-appearance: none;
  width: 100%;
  margin: 4.85px 0;
}
input[type=range]:focus {
  outline: none;
}
input[type=range]::-webkit-slider-runnable-track {
  width: 100%;
  height: 6.3px;
  cursor: pointer;
  box-shadow: 0px 0px 0px #92fff7, 0px 0px 0px #acfff9;
  background: #6b6c6d;
  border: 0px solid #cb85cd;
}
input[type=range]::-webkit-slider-thumb {
  box-shadow: 0px 0px 2.6px #000000, 0px 0px 0px #0d0d0d;
  border: 0px solid #000000;
  height: 16px;
  width: 16px;
  border-radius: 50px;
  background: #ffffff;
  cursor: pointer;
  -webkit-appearance: none;
  margin-top: -4.85px;
}
input[type=range]:focus::-webkit-slider-runnable-track {
  background: #78797a;
}
input[type=range]::-moz-range-track {
  width: 100%;
  height: 6.3px;
  cursor: pointer;
  box-shadow: 0px 0px 0px #92fff7, 0px 0px 0px #acfff9;
  background: #6b6c6d;
  border-radius: 25px;
  border: 0px solid #cb85cd;
}
input[type=range]::-moz-range-thumb {
  box-shadow: 0px 0px 2.6px #000000, 0px 0px 0px #0d0d0d;
  border: 0px solid #000000;
  height: 16px;
  width: 16px;
  border-radius: 50px;
  background: #ffffff;
  cursor: pointer;
}
input[type=range]::-ms-track {
  width: 100%;
  height: 6.3px;
  cursor: pointer;
  background: transparent;
  border-color: transparent;
  color: transparent;
}
input[type=range]::-ms-fill-lower {
  background: #5e5f60;
  border: 0px solid #cb85cd;
  border-radius: 50px;
  box-shadow: 0px 0px 0px #92fff7, 0px 0px 0px #acfff9;
}
input[type=range]::-ms-fill-upper {
  background: #6b6c6d;
  border: 0px solid #cb85cd;
  border-radius: 50px;
  box-shadow: 0px 0px 0px #92fff7, 0px 0px 0px #acfff9;
}
input[type=range]::-ms-thumb {
  box-shadow: 0px 0px 2.6px #000000, 0px 0px 0px #0d0d0d;
  border: 0px solid #000000;
  height: 16px;
  width: 16px;
  border-radius: 50px;
  background: #ffffff;
  cursor: pointer;
  height: 6.3px;
}
input[type=range]:focus::-ms-fill-lower {
  background: #6b6c6d;
}
input[type=range]:focus::-ms-fill-upper {
  background: #78797a;
}

.video-progress-bar {
  position: absolute;
  bottom: 87%;
}
</style>