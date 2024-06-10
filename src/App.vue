<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { Ref } from 'vue'

const mode = ref("single")
const cameras: Ref<{ id: string, label: string }[]> = ref([])
const opacity = ref(0.5)
// Reference to html elements
const video = ref()
const video0 = ref()
const video1 = ref()
const video2 = ref()
const video3 = ref()
const images: Ref<{ name: string, data: string | ArrayBuffer | null }[]> = ref([])
const imageData = ref(null)

const width = 1920
const height = 1080
const gridTemplate = `repeat(2, ${height / 2}px) / repeat(2, ${width / 2}px)`

let localStream: any = null

//================================
// Methods
//================================

const stopVideo = (video: any) => {
  video.value.pause()
  video.value.srcObject = null
  video.value.src = ""
}

function stop() {
  stopVideo(video)
  stopVideo(video0)
  stopVideo(video1)
  stopVideo(video2)
  stopVideo(video3)
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

const setImage = (data: any) => {
  imageData.value = data
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
    video1.value.srcObject = stream
    video2.value.srcObject = stream
    video3.value.srcObject = stream
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
      setImage(url)
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
      setImage(images.value[0].data)
    }
  }
}

const selectImage = (e: any) => {
  const idx = e.target.value
  setImage(images.value[idx].data)
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
  <div>
    <!-- Mode -->
    <label>
      Mode
      <label><input type="radio" v-model="mode" value="single"> Single</label>
      <label><input type="radio" v-model="mode" value="quad"> Quad</label>
    </label>
    <br>
    <!-- Opacity of image -->
    <label>
      Opacity
      <input type="range" min="0" max="1" step="0.01" v-model.number="opacity">
      <button @click="opacity = 0.0">0</button>
      <button @click="opacity = 0.5">0.5</button>
      <button @click="opacity = 1.0">1</button>
    </label>
    <br>
    <!-- Camera list -->
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
    <!-- Stop camera -->
    <button @click="stop"><i class="bi bi-stop-fill"></i></button>
    <br>
    <!-- Image load -->
    <input type="file" @change="loadLocalImages" multiple /><br />
    <!-- Local storage -->
    <button @click="save">Save</button>
    <button @click="load">Load</button>
    <select @change="selectImage">
      <template v-for="(image, idx) in images">
        <option :value="idx">{{ image.name }}</option>
      </template>
    </select>
  </div>

  <!-- Stage -->
  <div>
    <!-- Single view -->
    <div v-show="mode === 'single'" class="container">
      <!-- Camera -->
      <div><video ref="video" :width :height autoplay="true"></video></div>
      <!-- Photo-->
      <div><img v-if="imageData" :src="imageData" :width :height></div>
    </div>
    <!-- Quad view -->
    <div v-show="mode === 'quad'" class="quad">
      <div class="half-container">
        <div><video ref="video0" :width :height autoplay="true"></video></div>
        <div><img v-if="imageData" :src="imageData" :width :height></div>
      </div>
      <div class="half-container">
        <div><video ref="video1" :width :height autoplay="true"></video></div>
        <div><img v-if="imageData" :src="imageData" :width :height></div>
      </div>
      <div class="half-container">
        <div><video ref="video2" :width :height autoplay="true"></video></div>
        <div><img v-if="imageData" :src="imageData" :width :height></div>
      </div>
      <div class="half-container">
        <div><video ref="video3" :width :height autoplay="true"></video></div>
        <div><img v-if="imageData" :src="imageData" :width :height></div>
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

.half-container {
  width: 960px;
  height: 540px;
  position: relative
}

.container div,
.half-container div {
  position: absolute
}

.quad {
  display: grid;
  grid-template: v-bind("gridTemplate");
}

.quad>div {
  overflow: hidden;
}

img {
  opacity: v-bind("opacity");
}
</style>