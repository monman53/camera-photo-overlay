<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import type { Ref } from 'vue'

const mode = ref("single")
const cameras: Ref<{ id: string, label: string }[]> = ref([])
const opacity = ref(0.5)
const globalScale = ref(3.0)
// Reference to html elements
const video = ref()
const video0 = ref()
const video1 = ref()
const video2 = ref()
const video3 = ref()
const images: Ref<any[]> = ref([]) // TODO: type
const image: Ref<any> = ref(null)
const imageData = ref(null)

const width = 1920
const halfWidth = width / 2
const height = 1080
const halfHeight = height / 2
const gridTemplate = `repeat(2, ${height / 2}px) / repeat(2, ${width / 2}px)`

let localStream: any = null

const quadStyles = computed(() => {
  const styles: any[] = []
  if (image.value) {
    for (const q of image.value.quad) {
      const scale = globalScale.value
      const tx = q.cx - halfWidth / scale / 2
      const ty = q.cy - halfHeight / scale / 2
      const style = {
        transformOrigin: `${tx}px ${ty}px`,
        transform: `matrix(${scale}, 0, 0, ${scale}, ${-tx}, ${-ty})`
      }
      styles.push(style)
    }
  }
  return styles
})

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

const setImage = (img: any) => {
  image.value = img
  imageData.value = img.data
}

function start(e: any) {
  const id = e.target.value

  // if (id === "") {
  //   stop()
  //   return
  // }

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
      const url = reader.result;
      image.value = {
        name: file.name,
        data: url,
        quad: [
          { cx: halfWidth / 2, cy: halfHeight / 2, scale: 2 },
          { cx: halfWidth + halfWidth / 2, cy: halfHeight / 2, scale: 2 },
          { cx: halfWidth / 2, cy: halfHeight + halfHeight / 2, scale: 2 },
          { cx: halfWidth + halfWidth / 2, cy: halfHeight + halfHeight / 2, scale: 2 },
        ]
      }
      images.value.push(image.value)
      setImage(image.value)
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
      setImage(images.value[0])
    }
  }
}

const clear = () => {
  if (window.confirm("Really clear?")) {
    localStorage.removeItem('images')
  }
}

const download = () => {
  const text = JSON.stringify(images.value);
  const blob = new Blob([text], { type: 'text/plain' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'data.txt';
  a.click();
  URL.revokeObjectURL(url);
}

const upload = (e: any) => {
  const file = e.target.files[0];
  if (file && file.type === 'text/plain') {
    const reader = new FileReader();
    reader.onload = function (e_: any) {
      const text = e_.target.result;
      images.value = JSON.parse(text)
      setImage(images.value[0])
    };
    reader.readAsText(file);
  } else {
    alert('Please upload a valid text file.');
  }
}

const selectImage = (e: any) => {
  const idx = e.target.value
  setImage(images.value[idx])
}

//================================
// Hooks
//================================

onMounted(() => {
  updateCameras()
})

const svg = ref()

const getPositionOnSvg = (clientX: number, clientY: number) => {
  const rect = svg.value.getBoundingClientRect()
  const x = clientX - rect.left
  const y = clientY - rect.top
  return [x, y]
}

// Event handlers
let moveHandler: any = null;
const svgMoveStartHandler = (e: any, quad: any) => {
  // e.preventDefault();
  const [x0, y0] = getPositionOnSvg(e.clientX, e.clientY);
  const [cx0, cy0] = [quad.cx, quad.cy];
  const handler = (e_: any) => {
    e_.preventDefault();
    let clientX = e_.clientX
    let clientY = e_.clientY
    if (e_.type == 'touchmove') {
      clientX = e_.touches[0].clientX
      clientY = e_.touches[0].clientY
    }
    const [x, y] = getPositionOnSvg(clientX, clientY);
    const dx = (x - x0)
    const dy = (y - y0)
    quad.cx = cx0 + dx
    quad.cy = cy0 + dy
  }
  moveHandler = handler;
}
const svgMoveHandler = (e: any) => {
  // e.preventDefault();
  if (moveHandler !== null) {
    moveHandler(e)
  }
}
const svgMoveEndHandler = () => {
  moveHandler = null;
  // tpCache = [];
}

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
    <label>
      Scale
      <input type="range" min="1" max="10" step="0.01" v-model.number="globalScale">
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
    <button @click="clear">Clear</button>
    <button @click="download">download</button>
    <input type="file" @change="upload" />
  </div>

  <!-- Stage -->
  <div>
    <!-- Single view -->
    <div v-show="mode === 'single'" class="container">
      <!-- Camera -->
      <div><video ref="video" :width :height autoplay="true"></video></div>
      <!-- Photo-->
      <div><img v-if="imageData" :src="imageData" :width :height></div>
      <!-- SVG -->
      <div>
        <svg :width :height ref="svg" @pointermove="svgMoveHandler" @touchmove="svgMoveHandler"
          @pointerup="svgMoveEndHandler">
          <g v-if="image">
            <g v-for="q of image.quad">
              <rect :x="q.cx - halfWidth / 2 / globalScale" :y="q.cy - halfHeight / 2 / globalScale"
                :width="halfWidth / globalScale" :height="halfHeight / globalScale" fill="transparent" stroke="red"
                stroke-width="2" @pointerdown="svgMoveStartHandler($event, q)"></rect>
            </g>
          </g>
        </svg>
      </div>
    </div>
    <!-- Quad view -->
    <div v-show="mode === 'quad'" class="quad" @click="opacity = 1 - Math.floor(opacity)">
      <div class="half-container">
        <div :style="quadStyles[0]">
          <div><video ref="video0" :width :height autoplay="true"></video></div>
          <div><img v-if="imageData" :src="imageData" :width :height></div>
        </div>
      </div>
      <div class="half-container">
        <div :style="quadStyles[1]">
          <!-- <div> -->
          <div><video ref="video1" :width :height autoplay="true"></video></div>
          <div><img v-if="imageData" :src="imageData" :width :height></div>
        </div>
      </div>
      <div class="half-container">
        <div :style="quadStyles[2]">
          <div><video ref="video2" :width :height autoplay="true"></video></div>
          <div><img v-if="imageData" :src="imageData" :width :height></div>
        </div>
      </div>
      <div class="half-container">
        <div :style="quadStyles[3]">
          <div><video ref="video3" :width :height autoplay="true"></video></div>
          <div><img v-if="imageData" :src="imageData" :width :height></div>
        </div>
      </div>
    </div>
  </div>
  <!-- Image list -->
  <div>
    <template v-for="image in images">
      <img :src="image.data" width="256" @click="setImage(image)" style="border: solid green">
    </template>
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
  position: relative;
  /* border: solid 1px; */
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