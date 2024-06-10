<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { Ref } from 'vue'

const cameras: Ref<{ id: string, label: string }[]> = ref([])
const video = ref()
const opacity = ref(0.5)

let localStream: any = null

//================================
// Methods
//================================

function stop() {
  video.value.pause()
  video.value.srcObject = null
  video.value.src = ""

  if (localStream && localStream.getTracks) {
    const tracks = localStream.getTracks();
    for (const idx in tracks) {
      tracks[idx].stop()
    }
  }
  localStream = null
}

function updateCameras() {
  cameras.value = []
  navigator.mediaDevices.enumerateDevices()
    .then(function (devices) {
      devices.forEach(function (device) {
        if (device.kind === 'videoinput') {
          var id = device.deviceId;
          var label = device.label || "camera"; // label is available for https 
          cameras.value.push({ id, label });
        }
      });
    })
    .catch(function (err) {
      console.error('enumerateDevise ERROR:', err);
    });
}

function start(e: any) {
  const id = e.target.value

  if (id === "") {
    stop()
    return
  }

  var constraints = {
    video: {
      deviceId: id,
      width: 1920,
      height: 1080
    }
  };
  console.log('mediaDevice.getMedia() constraints:', constraints);

  navigator.mediaDevices.getUserMedia(
    constraints
  ).then(function (stream) {
    localStream = stream;
    video.value.srcObject = stream
  }).catch(function (err) {
    console.error('getUserMedia Err:', err);
  });
}

//================================
// Hooks
//================================

onMounted(() => {
  updateCameras()
})

</script>

<template>
  <!-- Controller -->
  <label>
    Camera list:
    <select @change="start">
      <template v-for="camera in cameras">
        <option value="">-- select --</option>
        <option :value="camera.id">{{ camera.label }}</option>
      </template>
    </select>
  </label>
  <button @click="updateCameras"><i class="bi bi-arrow-clockwise"></i></button>
  <button @click="stop"><i class="bi bi-stop-fill"></i></button>
  <label>
    Opacity
    <input type="range" min="0" max="1" step="0.01" v-model.number="opacity">
    <button @click="opacity=0.0">0</button>
    <button @click="opacity=0.5">0.5</button>
    <button @click="opacity=1.0">1</button>
  </label>

  <!-- Stage -->
  <div class="container">
    <!-- Camera -->
    <div>
      <video ref="video" id="local_video" width="1920px" height="1080px" autoplay="true"
        style="border: 1px solid;"></video>
    </div>
    <!-- Photo-->
    <div>
      <img width="1920" height="1080" src="https://raw.githubusercontent.com/monman53/juliaset/master/screenshot.png">
    </div>
  </div>
</template>

<style scoped>
.container {
  position: relative
}

.container>div {
  position: absolute
}

img {
  opacity: v-bind("opacity");
}
</style>