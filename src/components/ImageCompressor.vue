<template>
  <div class="container">
    <h1>Image Compressor</h1>
    <div
      class="drop-area"
      @click="triggerFileSelect"
      @dragover.prevent
      @drop.prevent="handleDrop"
    >
      <input
        type="file"
        accept="image/*"
        ref="fileInput"
        class="file-input"
        @change="handleFileChange"
      />
      <p v-if="!originalUrl">Drag & Drop image or click to select</p>
      <p v-else>{{ originalFile.name }}</p>
    </div>

    <div class="settings" v-if="originalFile">
      <label>
        Max Size (MB):
        <input type="number" v-model.number="maxSizeMB" min="0" step="0.1" />
      </label>
      <label>
        Max Width/Height:
        <input type="number" v-model.number="maxWidthOrHeight" min="0" />
      </label>
      <button @click="compressImage" :disabled="loading">Optimize</button>
    </div>

    <div class="preview" v-if="compressedUrl">
      <div>
        <h3>Before</h3>
        <img :src="originalUrl" alt="before" />
      </div>
      <div>
        <h3>After</h3>
        <img :src="compressedUrl" alt="after" />
      </div>
    </div>

    <div class="info" v-if="compressedFile">
      <p>Size before: {{ (originalSize/1024).toFixed(2) }} KB</p>
      <p>Size after: {{ (compressedSize/1024).toFixed(2) }} KB</p>
      <p>Saved: {{ savingPercent.toFixed(2) }}%</p>
      <button @click="download" v-if="!loading">Download</button>
      <div v-else class="spinner"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import imageCompression from 'browser-image-compression'

const fileInput = ref(null)
const originalFile = ref(null)
const compressedFile = ref(null)
const originalUrl = ref('')
const compressedUrl = ref('')
const maxSizeMB = ref(1)
const maxWidthOrHeight = ref(800)
const loading = ref(false)
const originalSize = ref(0)
const compressedSize = ref(0)

function triggerFileSelect() {
  fileInput.value.click()
}

function handleFileChange(e) {
  const file = e.target.files[0]
  if (file) {
    setFile(file)
  }
}

function handleDrop(e) {
  const file = e.dataTransfer.files[0]
  if (file) {
    setFile(file)
  }
}

function setFile(file) {
  originalFile.value = file
  originalUrl.value = URL.createObjectURL(file)
  compressedFile.value = null
  compressedUrl.value = ''
  originalSize.value = file.size
}

async function compressImage() {
  if (!originalFile.value) return
  loading.value = true
  try {
    const options = {
      maxSizeMB: maxSizeMB.value,
      maxWidthOrHeight: maxWidthOrHeight.value,
      useWebWorker: true,
    }
    const result = await imageCompression(originalFile.value, options)
    compressedFile.value = result
    compressedUrl.value = URL.createObjectURL(result)
    compressedSize.value = result.size
  } catch (err) {
    console.error(err)
  } finally {
    loading.value = false
  }
}

function download() {
  const a = document.createElement('a')
  a.href = compressedUrl.value
  a.download = compressedFile.value.name || 'compressed.jpg'
  a.click()
}

const savingPercent = computed(() => {
  if (!compressedSize.value || !originalSize.value) return 0
  return (1 - compressedSize.value / originalSize.value) * 100
})
</script>

<style scoped>
.container {
  text-align: center;
  max-width: 800px;
  margin: 0 auto;
}
.drop-area {
  border: 2px dashed #ccc;
  padding: 2rem;
  margin-bottom: 1rem;
  cursor: pointer;
}
.file-input {
  display: none;
}
.settings {
  margin-bottom: 1rem;
}
.settings label {
  margin-right: 1rem;
}
.preview {
  display: flex;
  justify-content: space-around;
  margin-top: 1rem;
}
.preview img {
  max-width: 100%;
  max-height: 300px;
}
.info {
  margin-top: 1rem;
}
.spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #555;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  animation: spin 1s linear infinite;
  margin: 0 auto;
}
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
