<template>
  <div
    class="w-[700px] h-[600px] bg-[#2e2e2e] px-4 pt-6 rounded-3xl drop-shadow-xl"
  >
    <div class="flex-col items-start justify-start flex rounded-lg w-max">
      <h1 class="text-white text-3xl font-semibold mb-2">
        {{ msg }}
      </h1>
      <p class="text-gray-400 text-sm mb-4">
        Used to classify types of vehicles registered in Indonesia
      </p>
    </div>

    <!-- Upload Area -->
    <div
      @drop.prevent="handleDrop"
      @dragover.prevent="isDragging = true"
      @dragleave.prevent="isDragging = false"
      @dragenter.prevent="isDragging = true"
      :class="[
        'border-2 border-dashed rounded-lg p-8 text-center transition-all duration-200 h-[calc(100%-10.5rem)]',
        isDragging
          ? 'border-blue-400 bg-blue-500/10'
          : 'border-gray-500 hover:border-gray-400',
      ]"
    >
      <input
        ref="fileInput"
        type="file"
        @change="handleFileSelect"
        accept="image/png,image/jpeg,image/jpg,image/webp,image/jfif"
        class="hidden"
      />

      <!-- Preview gambar -->
      <div
        v-if="image && !prediction"
        class="h-full flex flex-col items-center justify-center"
      >
        <div class="relative group">
          <img
            :src="image.preview"
            :alt="image.file.name"
            class="max-w-[200px] max-h-[200px] object-contain rounded-lg"
          />
          <button
            @click="removeImage"
            type="button"
            class="absolute top-2 right-2 bg-red-500 text-white p-2 rounded-full opacity-0 group-hover:opacity-100 transition-opacity"
          >
            <svg
              class="w-5 h-5"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              />
            </svg>
          </button>
        </div>
        <p class="text-white text-sm mt-3">{{ image.file.name }}</p>
        <p class="text-gray-400 text-xs">{{ formatBytes(image.file.size) }}</p>

        <button
          @click="$refs.fileInput.click()"
          type="button"
          class="mt-4 px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors"
        >
          Change Image
        </button>
      </div>

      <!-- Hasil Prediksi -->
      <div
        v-else-if="prediction"
        class="h-full flex flex-col items-center justify-center"
      >
        <div class="relative group mb-4">
          <img
            :src="image.preview"
            :alt="image.file.name"
            class="max-w-[200px] max-h-[200px] object-contain rounded-lg"
          />
        </div>

        <!-- Hasil -->
        <div class="bg-[#2a2a2a] rounded-lg p-4 w-full max-w-md">
          <div class="text-center mb-3">
            <p class="text-gray-400 text-sm mt-2">Classification Result:</p>
            <p class="text-white text-2xl font-bold mt-1">
              {{ prediction.class }}
            </p>
            <p v-if="prediction.confidence" class="text-blue-400 text-sm mt-1">
              Confidence: {{ (prediction.confidence * 100).toFixed(2) }}%
            </p>
          </div>

          <!-- Top 3 predictions -->
          <div v-if="top3" class="mt-3 space-y-2">
            <p class="text-gray-400 text-xs mb-2">Top 3 Predictions:</p>
            <div
              v-for="(pred, idx) in top3"
              :key="idx"
              class="flex justify-between items-center text-sm"
            >
              <span class="text-gray-300">{{ idx + 1 }}. {{ pred.class }}</span>
              <span class="text-blue-400">
                {{ (pred.confidence * 100).toFixed(1) }}%
              </span>
            </div>
          </div>
        </div>

        <button
          @click="reset"
          type="button"
          class="mt-4 px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors"
        >
          Classify again
        </button>
      </div>

      <!-- Area drag & drop -->
      <div
        v-else
        class="flex flex-col items-center justify-center h-full space-y-4"
      >
        <!-- Icon -->
        <svg
          class="w-20 h-20 text-gray-400"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
          />
        </svg>

        <!-- Text -->
        <div>
          <p class="text-lg font-medium text-white">
            {{
              isDragging
                ? "Drop it like it's hot!"
                : "Drag & drop the image here"
            }}
          </p>
          <p class="text-sm text-gray-400 mt-1">atau</p>
          <button
            @click="$refs.fileInput.click()"
            type="button"
            class="mt-3 px-6 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors font-medium"
          >
            Choose Image
          </button>
        </div>

        <p class="text-xs text-gray-400">
          Format: PNG, JPG, JPEG, WebP, JFIF • Max: 5MB
        </p>
      </div>
    </div>

    <!-- Button Classify -->
    <div class="w-full mt-2 items-end justify-end flex rounded-lg pt-2">
      <button
        @click="classify"
        :disabled="!image || isLoading || prediction !== null"
        :class="[
          'px-4 py-2 rounded-md transition-all',
          !image || isLoading || prediction !== null
            ? 'bg-gray-600 cursor-not-allowed'
            : 'bg-blue-500 hover:bg-blue-600',
        ]"
      >
        <span v-if="isLoading" class="flex items-center gap-2">
          <svg
            class="animate-spin h-4 w-4 text-white"
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
          Classifying...
        </span>
        <span v-else>Classify</span>
      </button>
    </div>

    <!-- Error Message -->
    <div
      v-if="error"
      class="mt-2 p-3 bg-red-500/20 border border-red-500 rounded-lg"
    >
      <p class="text-red-400 text-sm">{{ error }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

defineProps({
  msg: {
    type: String,
    default: "Upload Gambar",
  },
});

const API_URL = "http://localhost:8000"; // Sesuaikan dengan backend URL

const isDragging = ref(false);
const image = ref(null);
const fileInput = ref(null);
const maxSize = 5242880; // 5MB

const isLoading = ref(false);
const prediction = ref(null);
const top3 = ref(null);
const error = ref(null);

const handleDrop = (e) => {
  isDragging.value = false;
  const droppedFiles = Array.from(e.dataTransfer.files);
  if (droppedFiles.length > 0) {
    processFile(droppedFiles[0]);
  }
};

const handleFileSelect = (e) => {
  const selectedFiles = Array.from(e.target.files);
  if (selectedFiles.length > 0) {
    processFile(selectedFiles[0]);
  }
};

const processFile = (file) => {
  // Reset prediction dan error
  prediction.value = null;
  top3.value = null;
  error.value = null;

  // Validasi tipe file
  if (!file.type.startsWith("image/")) {
    error.value = `${file.name} bukan file gambar`;
    return;
  }

  // Validasi ukuran file
  if (file.size > maxSize) {
    error.value = `${file.name} terlalu besar. Maksimal 5MB`;
    return;
  }

  // Preview gambar
  const reader = new FileReader();
  reader.onload = (e) => {
    image.value = {
      file: file,
      preview: e.target.result,
    };
  };
  reader.readAsDataURL(file);
};

const removeImage = () => {
  image.value = null;
  prediction.value = null;
  top3.value = null;
  error.value = null;
  fileInput.value.value = "";
};

const reset = () => {
  removeImage();
};

const classify = async () => {
  if (!image.value) return;

  isLoading.value = true;
  error.value = null;

  try {
    const formData = new FormData();
    formData.append("file", image.value.file);

    const response = await axios.post(`${API_URL}/predict`, formData, {
      headers: {
        "Content-Type": "multipart/form-data",
      },
    });

    if (response.data.success) {
      prediction.value = response.data.prediction;
      top3.value = response.data.top_3;
    } else {
      error.value = "Prediction failed";
    }
  } catch (err) {
    console.error("Error:", err);
    error.value =
      err.response?.data?.detail ||
      "Failed to classify image. Please try again.";
  } finally {
    isLoading.value = false;
  }
};

const formatBytes = (bytes) => {
  if (!bytes) return "0 Bytes";

  const k = 1024;
  const sizes = ["Bytes", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));

  return Math.round((bytes / Math.pow(k, i)) * 100) / 100 + " " + sizes[i];
};
</script>
