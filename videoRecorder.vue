<template>
  <div>
      <template v-if="!isFinalVideo">
        <div v-show="!isPlayback">
          <div class="video-recorder-container">
            <b-row class="mb-0 mr-0 ml-0 mt-3 details-container">
              <b-col></b-col>
              <b-col>
                <div class="time-container text-center">
                  <span class="text-white">{{ timeTracker(currentTime) }}</span>
                </div>
              </b-col>
              <b-col class="text-right">
                <span class="text-danger record-container" v-if="isRecording">
                  <strong class="mr-2">REC</strong>
                  <i class="fa fa-circle"></i>
                </span>
                <b-link class="text-white ml-2 b-link-close" @click="closeVideoRecorder">
                  <i class="fa fa-times"></i>
                </b-link>
              </b-col>
            </b-row>
            <div class="video-container">
              <video ref="videoRecording" muted playsinline autoplay duration></video>
            </div>
            <b-row class="mt-0 mr-0 ml-0 mb-3 controls-container">
              <b-col></b-col>
              <b-col>
                <div class="circle-container">
                  <div @click="onStartRecording" class="circle-item" v-if="!isRecording"></div>
                  <div @click="onStopRecording" class="square-item" v-else></div>
                </div>
              </b-col>
              <b-col></b-col>
            </b-row>
          </div>
        </div>
        <div v-show="isPlayback">
          <div class="video-recorder-container">
            <b-row class="mb-0 mr-0 ml-0 mt-3 details-container">
              <b-col>
                <span
                  class="back-container text-white text-left"
                  @click="revertRecording"
                >&larr; Back</span>
              </b-col>
              <b-col></b-col>
              <b-col class="text-right">
                <span class="back-container text-white" @click="nextStep">Next &rarr;</span>
                <b-link class="text-white ml-2 b-link-close" @click="closeVideoRecorder">
                  <i class="fa fa-times"></i>
                </b-link>
              </b-col>
            </b-row>
            <div class="video-container">
              <video ref="videoPlayback" autoplay playsinline controls></video>
            </div>
          </div>
        </div>
      </template>
      <template v-else>
        <video ref="videoFinal" controls autoplay></video>
      </template>
  </div>
</template>

<script>
import forEach from "lodash/forEach";

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
      constraintObject: {
        audio: {
          echoCancellation: {
            exact: true
          },
          noiseSuppression: {
            exact: true
          }
        },
        video: true
      },
      navigator: navigator,
      chunks: [],
      isRecording: false,
      isPlayback: false,
      mediaRecorder: null,
      videoRecording: null,
      videoPlayback: null,
      currentTime: 0,
      interval: null,
      videoSrc: null,
      isFinalVideo: false,
      videoData: new FormData(),
      isVisible: false,
      mediaStreamObject: null
    };
  },
  methods: {
    closeVideoRecorder() {
    },
    onCreated() {
      this.videoRecording = this.$refs.videoRecording;
      this.videoPlayback = this.$refs.videoPlayback;
      this.navigator = navigator;
      this.isVisible = true;
      this.handleOlderBrowser();
      this.startStreaming();
    },
    onDestroyed() {
      this.stopStreaming();
      this.onStopRecording();
      this.destroyVideoPlayback();
      this.controlVisibility(false, false);
      this.isFinalVideo = false;
      this.videoRecording = null;
      this.videoPlayback = null;
      this.mediaRecorder = null;
      this.navigator = null;
      this.chunks = [];
      this.mediaStreamObject = null;
    },
    nextStep() {
      // this.$refs.videoFinal.src = this.videoPlayback.src;
      this.controlVisibility(false, false);
      this.isFinalVideo = false;
    },
    incrementTime() {
      this.currentTime = this.currentTime + 1;
    },
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
    revertRecording() {
      this.destroyVideoPlayback();
      this.startStreaming();
      this.controlVisibility(false, false);
    },
    destroyVideoPlayback() {
      this.videoPlayback.pause();
      this.videoPlayback.removeAttribute("src");
      this.videoPlayback.load();
    },
    controlVisibility(isRecording, isPlayback) {
      this.isRecording = isRecording;
      this.isPlayback = isPlayback;
    },
    onStartRecording() {
      if (this.mediaRecorder.state === "inactive") {
        this.mediaRecorder.start();
        this.interval = setInterval(this.incrementTime, 1000);
        this.controlVisibility(true, false);
      }
    },
    onStopRecording() {
      if (this.mediaRecorder.state === "recording") {
        this.mediaRecorder.stop();
        clearInterval(this.interval);
        this.interval = null;
        this.controlVisibility(false, true);
      }
    },
    onDataAvailable() {
      this.mediaRecorder.ondataavailable = event => {
        this.chunks.push(event.data);
      };
    },
    onStop() {
      this.mediaRecorder.onstop = event => {
        this.stopStreaming();
        if (this.isVisible) {
          let blob = new Blob(this.chunks, { type: "video/mp4" });
          let videoURL = window.URL.createObjectURL(blob);
          this.videoPlayback.src = videoURL;
          let file = new File([blob], `${new Date()}-crm-machaik`, {
            lastModified: new Date(),
            type: blob.type
          });
          this.videoData.append(file.name, file);
          this.chunks = [];
        }
      };
    },
    onPlay() {
      this.videoRecording.onloadedmetadata = event => {
        this.videoRecording.play();
      };
    },
    initializeTime() {
      this.currentTime = 0;
    },
    handleOlderBrowser() {
      if (this.navigator.mediaDevices === undefined) {
        this.navigator.mediaDevices = {};
        this.navigator.mediaDevices.getUserMedia = constraintObject => {
          let getUserMedia =
            this.navigator.webkitaGetUserMedia ||
            this.navigator.mozGetUserMedia;
          if (!getUserMedia) {
            return Promise.reject(
              new Error("getUserMedia is not implemented in this browser")
            );
          }
          return new Promise(resolve, reject => {
            getUsermedia.call(
              this.navigator,
              constraintObject,
              resolve,
              reject
            );
          });
        };
      }
    },
    startStreaming() {
      this.initializeTime();
      this.navigator.mediaDevices
        .getUserMedia(this.constraintObject)
        .then(mediaStreamObject => {
          this.mediaStreamObject = mediaStreamObject;
          this.mediaRecorder = new MediaRecorder(mediaStreamObject);
          if ("srcObject" in this.videoRecording) {
            this.videoRecording.srcObject = mediaStreamObject;
          } else {
            this.videoRecording.src = window.URL.createObjectURL(
              mediaStreamObject
            );
          }
          this.onPlay();
          this.onDataAvailable();
          this.onStop();
        })
        .catch(error => {
          console.log(error.name, error.message);
        });
    },
    stopStreaming() {
      if (!this.mediaStreamObject) return false;
      forEach(this.mediaStreamObject.getTracks(), track => {
        track.stop();
      });
    }
  },
  watch: {
    isPlayback(nv) {
      if (!nv) {
        this.mediaRecorder = null;
      }
    }
  },
  mounted() {
    this.onCreated();
  },
  beforeDestroy() {
    this.onDestroyed();
  }
};
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

.controls-container {
  position: absolute;
  width: 100%;
  bottom: 0;
}

.details-container {
  position: absolute;
  width: 100%;
  top: 0;
  z-index: 50;
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
</style>