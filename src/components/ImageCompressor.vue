<template>
    <div class="bg-slate-50 rounded-2xl font-sans">
        <div class="container mx-auto px-4 py-8 md:py-12">
            <header class="text-center mb-10">
                <h1 class="text-4xl font-bold text-slate-800">
                    Image Optimizer
                </h1>
                <p class="text-lg text-slate-600 mt-2">
                    Compress and resize your images in the browser.
                </p>
            </header>

            <div class="max-w-2xl mx-auto flex flex-col gap-8">
                <!-- 1. Upload Area -->
                <div class="rounded-2xl shadow-md bg-white p-6">
                    <div
                        class="border-2 border-dashed border-gray-200 rounded-xl text-center cursor-pointer hover:border-blue-500 hover:bg-blue-50 transition-all p-8"
                        @click="triggerFileSelect"
                        @dragover.prevent
                        @drop.prevent="handleDrop"
                    >
                        <input
                            type="file"
                            accept="image/*"
                            ref="fileInput"
                            class="hidden"
                            @change="handleFileChange"
                        />
                        <div
                            class="flex flex-col items-center justify-center text-slate-500"
                        >
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                width="20"
                                height="22"
                                id="upload"
                            >
                                <g
                                    fill="none"
                                    fill-rule="evenodd"
                                    stroke="#000"
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                >
                                    <path
                                        d="M1 16v3a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-3M14 5l-4-4-4 4M10 1v14"
                                    ></path>
                                </g>
                            </svg>
                            <p v-if="!originalFile" class="font-semibold">
                                Drag & drop file or
                                <span class="text-blue-600"
                                    >click to browse</span
                                >
                            </p>
                            <p v-else class="font-semibold text-slate-700">
                                {{ originalFile.name }}
                            </p>
                            <p class="text-sm mt-1">Max file size: 10MB</p>
                        </div>
                    </div>
                </div>

                <!-- 2. Settings Panel & Optimize Button -->
                <div
                    class="rounded-2xl shadow-md p-6 bg-white"
                    v-if="originalFile && showSettings"
                >
                    <h3 class="text-xl font-semibold text-slate-800 mb-5">
                        Settings
                    </h3>
                    <div class="space-y-5">
                        <!-- Max Size -->
                        <div>
                            <label
                                for="maxSize"
                                class="block text-sm font-medium text-slate-700 mb-1"
                                >Max Size</label
                            >
                            <div class="flex items-center">
                                <input
                                    id="maxSize"
                                    type="number"
                                    v-model.number="maxSize"
                                    min="0"
                                    step="0.1"
                                    class="w-full border border-gray-300 rounded-l-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500 z-10"
                                />
                                <select
                                    v-model="sizeUnit"
                                    class="-ml-px border border-gray-300 rounded-r-lg px-3 py-2 bg-slate-50 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
                                >
                                    <option value="KB">KB</option>
                                    <option value="MB">MB</option>
                                </select>
                            </div>
                        </div>
                        <!-- Resize -->
                        <div>
                            <div class="flex items-center justify-between mb-1">
                                <label
                                    class="text-sm font-medium text-slate-700"
                                    >Resize Image</label
                                >
                                <input
                                    type="checkbox"
                                    v-model="resize"
                                    class="h-5 w-5 rounded border-gray-300 text-blue-600 focus:ring-blue-500"
                                />
                            </div>
                            <input
                                type="number"
                                v-model.number="maxWidthOrHeight"
                                min="0"
                                class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-slate-100 disabled:cursor-not-allowed"
                                :disabled="!resize"
                                placeholder="e.g., 1920"
                            />
                        </div>
                    </div>
                    <div class="mt-6">
                        <button
                            @click="compressImage"
                            :disabled="loading || !originalFile"
                            class="w-full flex justify-center items-center px-4 py-3 bg-blue-600 text-white text-base font-semibold rounded-lg hover:bg-blue-700 disabled:bg-slate-400 transition-all"
                        >
                            <svg
                                v-if="loading"
                                class="animate-spin -ml-1 mr-3 h-5 w-5"
                                xmlns="http://www.w3.org/2000/svg"
                                fill="none"
                                viewBox="0 0 24 24"
                            >
                                <circle
                                    class="opacity-25"
                                    cx="12"
                                    cy="12"
                                    r="10"
                                    stroke="currentColor"
                                    stroke-width="4"
                                ></circle>
                                <path
                                    class="opacity-75"
                                    fill="currentColor"
                                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                                ></path>
                            </svg>
                            <span>{{
                                loading ? "Processing..." : "Optimize"
                            }}</span>
                        </button>
                    </div>
                </div>

                <!-- 3. Preview Block -->
                <div
                    class="rounded-2xl shadow-md p-6 bg-white"
                    v-if="loading || compressedUrl"
                >
                    <div
                        v-if="loading && !compressedUrl"
                        class="flex flex-col items-center justify-center h-full py-10"
                    >
                        <svg
                            class="animate-spin h-10 w-10 text-blue-600 mb-4"
                            xmlns="http://www.w3.org/2000/svg"
                            fill="none"
                            viewBox="0 0 24 24"
                        >
                            <circle
                                class="opacity-25"
                                cx="12"
                                cy="12"
                                r="10"
                                stroke="currentColor"
                                stroke-width="4"
                            ></circle>
                            <path
                                class="opacity-75"
                                fill="currentColor"
                                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                            ></path>
                        </svg>
                        <p class="text-slate-600 font-medium">
                            Compressing, please wait...
                        </p>
                    </div>

                    <div
                        v-if="compressedUrl"
                        class="grid grid-cols-1 sm:grid-cols-2 gap-4"
                    >
                        <div>
                            <h3
                                class="text-center font-semibold text-slate-700 mb-2"
                            >
                                Before
                            </h3>
                            <img
                                :src="originalUrl"
                                alt="Original"
                                class="rounded-lg border-2 border-gray-200 w-full object-contain"
                            />
                        </div>
                        <div>
                            <h3
                                class="text-center font-semibold text-slate-700 mb-2"
                            >
                                After
                            </h3>
                            <img
                                :src="compressedUrl"
                                alt="Compressed"
                                class="rounded-lg border-2 border-emerald-400 w-full object-contain"
                            />
                        </div>
                    </div>
                </div>

                <!-- 4. Info Block & Download Button -->
                <div
                    class="rounded-2xl shadow-md p-6 bg-white"
                    v-if="compressedFile"
                >
                    <div
                        class="grid grid-cols-1 sm:grid-cols-3 divide-y sm:divide-y-0 sm:divide-x divide-gray-200 text-center gap-y-4"
                    >
                        <div class="pt-4 sm:pt-0">
                            <p class="text-sm text-slate-600">Original</p>
                            <p class="font-semibold text-slate-800 text-lg">
                                {{ (originalSize / 1024).toFixed(1) }} KB
                            </p>
                        </div>
                        <div class="pt-4 sm:pt-0">
                            <p class="text-sm text-slate-600">Compressed</p>
                            <p class="font-semibold text-emerald-600 text-lg">
                                {{ (compressedSize / 1024).toFixed(1) }} KB
                            </p>
                        </div>
                        <div class="pt-4 sm:pt-0">
                            <p class="text-sm text-slate-600">Savings</p>
                            <p class="font-semibold text-emerald-600 text-lg">
                                {{ savingPercent.toFixed(1) }}%
                            </p>
                        </div>
                    </div>
                    <div class="mt-6">
                        <button
                            @click="download"
                            class="w-full px-4 py-3 bg-emerald-600 text-white text-base font-semibold rounded-lg hover:bg-emerald-700 transition-all"
                        >
                            Download Optimized Image
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed } from "vue";
import imageCompression from "browser-image-compression";

const fileInput = ref(null);
const originalFile = ref(null);
const compressedFile = ref(null);
const originalUrl = ref("");
const compressedUrl = ref("");
const maxSize = ref(1);
const sizeUnit = ref("MB");
const resize = ref(true);
const maxWidthOrHeight = ref(1920);
const loading = ref(false);
const originalSize = ref(0);
const compressedSize = ref(0);
const showSettings = ref(true);

function triggerFileSelect() {
    fileInput.value.click();
}

function handleFileChange(e) {
    const file = e.target.files[0];
    if (file) {
        setFile(file);
    }
}

function handleDrop(e) {
    const file = e.dataTransfer.files[0];
    if (file) {
        setFile(file);
    }
}

function setFile(file) {
    originalFile.value = file;
    originalUrl.value = URL.createObjectURL(file);
    compressedFile.value = null;
    compressedUrl.value = "";
    originalSize.value = file.size;
    showSettings.value = true;
}

async function compressImage() {
    if (!originalFile.value) return;
    loading.value = true;
    showSettings.value = false;
    compressedFile.value = null;
    compressedUrl.value = "";

    try {
        const options = {
            maxSizeMB:
                sizeUnit.value === "MB" ? maxSize.value : maxSize.value / 1024,
            useWebWorker: true,
        };
        if (resize.value) {
            options.maxWidthOrHeight = maxWidthOrHeight.value;
        }
        const result = await imageCompression(originalFile.value, options);
        compressedFile.value = result;
        compressedUrl.value = URL.createObjectURL(result);
        compressedSize.value = result.size;
    } catch (err) {
        console.error(err);
        alert(
            "Failed to compress image. Please check the console for details.",
        );
    } finally {
        loading.value = false;
    }
}

function download() {
    const a = document.createElement("a");
    a.href = compressedUrl.value;
    a.download = `compressed-${originalFile.value.name}`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
}

const savingPercent = computed(() => {
    if (!compressedSize.value || !originalSize.value) return 0;
    return (1 - compressedSize.value / originalSize.value) * 100;
});
</script>

<style>
/* No custom styles needed, everything is handled by Tailwind CSS utility classes. */
</style>
