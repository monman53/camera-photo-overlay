<script setup lang="ts">
import { ref } from 'vue'
import type { Ref } from 'vue'

const cameras: Ref<{ id: string, label: string }[]> = ref([])

const video = ref()
let localStream: any = null

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

//--------------------

function getCameras() {
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
</script>

<template>
  <button @click="getCameras">Update Camera List</button>
  <label>
    Camera list:
    <select @change="start">
      <template v-for="camera in cameras">
        <option value="">-- select --</option>
        <option :value="camera.id">{{ camera.label }}</option>
      </template>
    </select>
  </label>
  <button @click="stop">Stop</button>

  <div class="container">
    <div>
      <video ref="video" id="local_video" width="1920px" height="1080px" autoplay="true"
        style="border: 1px solid;"></video>
    </div>
    <div>
      <img width="1920" height="1080" src="https://raw.githubusercontent.com/monman53/juliaset/master/screenshot.png">
    </div>
  </div>
</template>

<style scoped></style>
