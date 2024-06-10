<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { Ref } from 'vue'

const cameras: Ref<{ id: string, label: string }[]> = ref([])
const opacity = ref(0.5)
// Reference to html elements
const video = ref()
const video0 = ref()
const img = ref()
const images: Ref<{ name: string, data: string | ArrayBuffer | null }[]> = ref([])

const width = 1920
const height = 1080

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
      width,
      height,
    }
  };
  console.log('mediaDevice.getMedia() constraints:', constraints);

  navigator.mediaDevices.getUserMedia(
    constraints
  ).then(function (stream) {
    localStream = stream;
    video.value.srcObject = stream
    video0.value.srcObject = stream
  }).catch(function (err) {
    console.error('getUserMedia Err:', err);
  });
}

const loadLocalImages = (e: any) => {
  const files = e.target.files;
  for (const file of files) {
    const reader = new FileReader();
    reader.onload = () => {
      let url = reader.result;
      img.value.src = url
      images.value.push({ name: file.name, data: url })
    }
    reader.readAsDataURL(file);
  }
}

const save = () => {
  localStorage.setItem('images', JSON.stringify(images.value))
}

const load = () => {
  const localData = localStorage.getItem('images')
  if (localData) {
    images.value = JSON.parse(localData)
    if (images.value.length > 0) {
      img.value.src = images.value[0].data
    }
  }
}

const selectImage = (e: any) => {
  const idx = e.target.value
  img.value.src = images.value[idx].data
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
    <button @click="opacity = 0.0">0</button>
    <button @click="opacity = 0.5">0.5</button>
    <button @click="opacity = 1.0">1</button>
  </label>
  <input type="file" @change="loadLocalImages" multiple /><br />
  <button @click="save">Save</button>
  <button @click="load">Load</button>
  <select @change="selectImage">
    <template v-for="(image, idx) in images">
      <option :value="idx">{{ image.name }}</option>
    </template>
  </select>

  <!-- Stage -->
  <div>
    <!-- Single view -->
    <div class="container">
      <!-- Camera -->
      <div>
        <video ref="video" :width :height autoplay="true" style="border: 1px solid;"></video>
      </div>
      <!-- Photo-->
      <div>
        <img ref="img" :width :height>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  width: 1920px;
  height: 1080px;
  position: relative
}

.container div {
  position: absolute
}

img {
  opacity: v-bind("opacity");
}
</style>