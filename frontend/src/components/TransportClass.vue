<template>
  <div
    class="w-[700px] h-[600px] bg-[#2e2e2e] p-4 py-6 rounded-3xl drop-shadow-xl"
  >
    <h1 class="text-white text-3xl font-semibold mb-4">{{ msg }}</h1>

    <!-- Upload Area -->
    <div
      @drop.prevent="handleDrop"
      @dragover.prevent="isDragging = true"
      @dragleave.prevent="isDragging = false"
      @dragenter.prevent="isDragging = true"
      :class="[
        'border-2 border-dashed rounded-lg p-8 text-center transition-all duration-200 h-[calc(100%-6.5rem)]',
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
        v-if="image"
        class="h-full flex flex-col items-center justify-center"
      >
        <div class="relative group">
          <img
            :src="image.preview"
            :alt="image.file.name"
            class="max-w-[350px] max-h-[350px] object-contain rounded-lg"
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
          Ganti Gambar
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
                ? "Lepaskan gambar di sini"
                : "Drag & drop gambar di sini"
            }}
          </p>
          <p class="text-sm text-gray-400 mt-1">atau</p>
          <button
            @click="$refs.fileInput.click()"
            type="button"
            class="mt-3 px-6 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors font-medium"
          >
            Pilih Gambar
          </button>
        </div>

        <p class="text-xs text-gray-400">
          Format: PNG, JPG, JPEG, WebP, JFIF • Max: 5MB per gambar
        </p>
      </div>
    </div>

    <div class="w-full my-2 items-end justify-end flex rounded-lg py-2">
      <button
        class="bg-blue-500 px-4 py-2 rounded-md hover:bg-blue-600 transition-colors"
      >
        Classify
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

defineProps({
  msg: {
    type: String,
    default: "Upload Gambar",
  },
});

const isDragging = ref(false);
const image = ref(null);
const fileInput = ref(null);
const maxSize = 5242880;

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
  // Validasi tipe file
  if (!file.type.startsWith("image/")) {
    alert(`${file.name} bukan file gambar`);
    return;
  }

  // Validasi ukuran file
  if (file.size > maxSize) {
    alert(`${file.name} terlalu besar. Maksimal 5MB`);
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
  fileInput.value.value = "";
};

const formatBytes = (bytes) => {
  if (!bytes) return "0 Bytes";

  const k = 1024;
  const sizes = ["Bytes", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));

  return Math.round((bytes / Math.pow(k, i)) * 100) / 100 + " " + sizes[i];
};

// Expose image untuk diakses dari parent component jika perlu
defineExpose({
  image,
  getFile: () => image.value?.file || null,
  clear: removeImage,
});
</script>
