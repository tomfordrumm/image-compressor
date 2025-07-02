<template>
  <div class="container mx-auto max-w-2xl p-4 text-center">
    <h1 class="text-2xl font-semibold mb-4">Image Compressor</h1>
    <div
      class="drop-area border-2 border-dashed border-gray-400 p-6 rounded cursor-pointer bg-gray-50 hover:bg-gray-100 transition-colors"
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
      <p v-if="!originalUrl" class="text-gray-500">Drag & Drop image or click to select</p>
      <p v-else class="text-gray-700">{{ originalFile.name }}</p>
    </div>

    <div class="settings flex flex-wrap items-end gap-4 mt-4" v-if="originalFile">
      <label class="flex items-center gap-2">
        <span>Max Size:</span>
        <input type="number" v-model.number="maxSize" min="0" step="0.1" class="border rounded px-2 py-1 w-24" />
        <select v-model="sizeUnit" class="border rounded px-2 py-1">
          <option value="KB">KB</option>
          <option value="MB">MB</option>
        </select>
      </label>
      <label class="flex items-center gap-2" v-if="resize">
        <span>Max Width/Height:</span>
        <input type="number" v-model.number="maxWidthOrHeight" min="0" class="border rounded px-2 py-1 w-24" />
      </label>
      <label class="flex items-center gap-2">
        <input type="checkbox" v-model="resize" /> Resize
      </label>
      <button @click="compressImage" :disabled="loading" class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 transition-colors">
        Optimize
      </button>
    </div>

    <div class="preview flex justify-around mt-4" v-if="compressedUrl">
      <div>
        <h3 class="font-medium mb-2">Before</h3>
        <img :src="originalUrl" alt="before" class="max-w-full max-h-72" />
      </div>
      <div>
        <h3 class="font-medium mb-2">After</h3>
        <img :src="compressedUrl" alt="after" class="max-w-full max-h-72" />
      </div>
    </div>

    <div class="info mt-4" v-if="compressedFile">
      <p>Size before: {{ (originalSize/1024).toFixed(2) }} KB</p>
      <p>Size after: {{ (compressedSize/1024).toFixed(2) }} KB</p>
      <p>Saved: {{ savingPercent.toFixed(2) }}%</p>
      <button @click="download" v-if="!loading" class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600 transition-colors">Download</button>
      <div v-else class="spinner w-6 h-6 border-4 border-gray-300 border-t-gray-700 rounded-full animate-spin mx-auto"></div>
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
const maxSize = ref(1)
const sizeUnit = ref('MB')
const resize = ref(false)
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
      maxSizeMB: sizeUnit.value === 'MB' ? maxSize.value : maxSize.value / 1024,
      useWebWorker: true,
    }
    if (resize.value) {
      options.maxWidthOrHeight = maxWidthOrHeight.value
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
.file-input {
  display: none;
}
</style>
