<template>
  <div class="camera h-100">
<!--    <h1 style="position: absolute; top: 40px; left: 40px; color: #fff">-->
<!--      {{ facingMode }}-->
<!--    </h1>-->

<!--    <div>-->
<!--      <div class="label">Pan:</div>-->
<!--      <input name="pan" type="range" disabled>-->
<!--    </div>-->
<!--    <div>-->
<!--      <div class="label">Tilt:</div>-->
<!--      <input name="tilt" type="range" disabled>-->
<!--    </div>-->
<!--    <div>-->
<!--      <div class="label">Zoom:</div>-->
<!--      <input name="zoom" type="range" disabled>-->
<!--    </div>-->

    <video ref="video" style="pointer-events: none"></video>



    <div class="camera__thumbnail" :style="thumbnailStyle" @click="downloadImage"></div>
    <button @click="captureImage" class="camera__btn-capture"></button>
    <button @click="flipCamera" class="camera__btn-flip">Flip</button>
    <canvas ref="canvas" class="camera__canvas"></canvas>
  </div>
</template>

<script>
const EXTENSION = "jpeg";

export default {
  name: "Camera",
  data() {
    return {
      mediaStream: null,
      devices: null,
      data: null,
      track: null,
      imageCapture: null,
      imageBase64: null,

      isFrontCamera: true,
    };
  },
  computed: {
    facingMode() {
      return this.isFrontCamera ? "user" : { exact: "environment" };
    },
    thumbnailStyle() {
      return {
        backgroundImage: `url(${this.imageBase64})`
      }
    }
  },
  async mounted() {
    this.devices = await navigator.mediaDevices.enumerateDevices();
    this.initCamera();
  },
  methods: {
    initCamera() {
      this.stopStream();

      const constraints = {
        audio: false,
        video: {
          facingMode: this.facingMode,
          width: { min: 1024, ideal: 1280, max: 1920 },
          height: { min: 576, ideal: 720, max: 1080 },
        },
      };

      navigator.mediaDevices
        .getUserMedia(constraints)
        .then((mediaStream) => {
          this.mediaStream = mediaStream
          this.$refs.video.srcObject = mediaStream;
          this.$refs.video.onloadedmetadata = () => this.$refs.video.play();
          mediaStream.onremovetrack = function () {
            console.log("Stream ended");
          };
        })
        .catch((err) => {
          console.log(err.name + ": " + err.message);
        });
    },
    flipCamera() {
      this.isFrontCamera = !this.isFrontCamera;
      this.initCamera();
    },
    stopStream() {
      if (!this.mediaStream) {
        return;
      }
      this.mediaStream.getTracks().forEach((track) => track.stop());
    },
    async captureImage() {
      this.$refs.canvas.width = this.$refs.video.videoWidth;
      this.$refs.canvas.height = this.$refs.video.videoHeight;
      this.$refs.canvas.getContext('2d')
        .drawImage(
          this.$refs.video, 0, 0, this.$refs.canvas.width, this.$refs.canvas.height
        );
      console.log(`image/${EXTENSION}`);
      this.imageBase64 = this.$refs.canvas.toDataURL(`image/${EXTENSION}`)

    },
    downloadImage() {
      const a = document.createElement("a");
      a.href = this.imageBase64;
      a.download = `${Date.now()}.${EXTENSION}`;
      a.click();
    }
  },
};

// https://webrtc.github.io/samples/
// https://www.html5rocks.com/en/tutorials/getusermedia/intro/
// https://www.twilio.com/blog/choosing-cameras-javascript-mediadevices-api-html
// https://stackoverflow.com/questions/28282385/webrtc-firefox-constraints/28911694#28911694
// https://developer.mozilla.org/en-US/docs/Web/API/MediaStream_Image_Capture_API
</script>

<style lang="scss">
.camera {
  position: relative;
  overflow: hidden;
  background-color: #000;
  &__btn {
    &-capture {
      $size: 70px;

      position: absolute;
      bottom: 30px;
      left: 50%;
      transform: translate(-50%, 0);
      height: $size;
      width: $size;
      border-radius: 50%;

      border-width: 10px;
      border-style: double;
      border-color: #ffffff;
      background-color: transparent;
    }
    &-flip {
      $size: 50px;

      position: absolute;
      bottom: 40px;
      right: 40px;
      height: $size;
      width: $size;
    }
  }
  &__canvas {
    display: none;
  }
  &__thumbnail {
    display: flex;
    align-items: center;
    justify-content: center;
    position: absolute;
    left: 40px;
    bottom: 40px;
    width: 50px;
    height: 50px;
    border-radius: 5px;
    background-size: cover;
    background-position: center;
    background-color: #282828;

  }
}
video {
  width: 100%;
}
video {
  height: 100%;
  object-fit: contain;
}
</style>
