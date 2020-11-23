<template>
  <div class="container">
    <div class="row">
      <div class="col-md-6">
        <h2>Current Camera</h2>
        <div class="border">
          <vue-web-cam
            ref="webcam"
            :device-id="deviceId"
            width="100%"
            @started="onStarted"
            @stopped="onStopped"
            @error="onError"
            @cameras="onCameras"
            @camera-change="onCameraChange"
          />
        </div>

        <div class="row">
          <div class="col-md-12" style="background-color: white;">
            <select v-model="camera">
              <option selected>-- Select Device --</option>
              <option
                v-for="device in devices"
                :key="device.deviceId"
                :value="device.deviceId"
              >
                {{ device.label }}
              </option>
            </select>
          </div>
          <div class="col-md-12">
            <v-btn @click="onCapture">
              Capture Photo
            </v-btn>
            <v-btn @click="onStop">
              Stop Camera
            </v-btn>
            <v-btn @click="onStart">
              Start Camera
            </v-btn>
          </div>
        </div>
      </div>
      <div class="col-md-6">
        <h2>Captured Image</h2>
        <figure class="figure">
          <img :src="img" class="img-responsive" />
        </figure>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'nuxt-property-decorator'
// import { WebCam } from 'vue-web-cam'

@Component({ components: {} })
class Sandbox extends Vue {
  '$refs': {
    webcam: any
  }
  img: any = null
  camera: any = null
  deviceId: any = null
  devices: any = []

  get device() {
    return this.devices.find(
      (n: { deviceId: any }) => n.deviceId === this.deviceId
    )
  }

  @Watch('camera')
  onCameraChanged(val: any) {
    this.deviceId = val
  }

  @Watch('devices')
  onDeviceChanged() {
    const first = this.devices[0]
    if (first) {
      this.camera = first.deviceId
      this.deviceId = first.deviceId
    }
  }
  onCapture() {
    this.img = this.$refs.webcam.capture()
  }
  onStarted(stream: any) {
    console.log('On Started Event', stream)
  }
  onStopped(stream: any) {
    console.log('On Stopped Event', stream)
  }
  onStop() {
    this.$refs.webcam.stop()
  }
  onStart() {
    console.log(this.$refs.webcam)
    this.$refs.webcam.start()
  }
  onError(error: any) {
    console.log('On Error Event', error)
  }
  onCameras(cameras: any) {
    this.devices = cameras
    console.log('On Cameras Event', cameras)
  }
  onCameraChange(deviceId: any) {
    this.deviceId = deviceId
    this.camera = deviceId
    console.log('On Camera Change Event', deviceId)
  }
}
export default Sandbox
</script>