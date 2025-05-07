<template>
  <div>
    <h2 class="text-xl font-semibold mb-6 text-gray-800">Document Scanner</h2>
    
    <div 
      @click="triggerFileInput"
      class="border-2 border-dashed border-gray-300 rounded-lg p-8 text-center cursor-pointer hover:border-green-500 transition-colors mb-6"
    >
      <p class="text-gray-500" style="margin-bottom: 0;">Click to upload a document image</p>
      <input 
        type="file" 
        ref="fileInput" 
        @change="handleFileUpload" 
        accept="image/*" 
        class="hidden"
      >
    </div>
    
    <div v-if="loading" class="text-center py-4">
      <div class="inline-block animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-green-600"></div>
      <p class="mt-2 text-gray-600">Processing...</p>
    </div>
    
    <div v-if="error" class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 mb-6">
      <p>{{ error }}</p>
    </div>
    
    <div v-if="originalImage || scannedImage" class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
      <div v-if="originalImage" class="bg-gray-50 p-4 rounded-lg">
        <h3 class="font-medium text-gray-700 mb-2">Original Image</h3>
        <img :src="originalImage" class="max-w-full h-auto rounded border border-gray-200">
      </div>
      <div v-if="scannedImage" class="bg-gray-50 p-4 rounded-lg">
        <h3 class="font-medium text-gray-700 mb-2">Scanned Document</h3>
        <img :src="scannedImage" class="max-w-full h-auto rounded border border-gray-200">
        <button 
          @click="downloadScannedImage"
          class="mt-3 bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-md transition-colors"
        >
          Download Scanned Document
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const fileInput = ref(null)
const originalImage = ref(null)
const scannedImage = ref(null)
const loading = ref(false)
const error = ref(null)

const triggerFileInput = () => fileInput.value.click()

const handleFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  try {
    loading.value = true
    error.value = null
    scannedImage.value = null
    
    originalImage.value = URL.createObjectURL(file)
    
    const formData = new FormData()
    formData.append('file', file)
    
    const response = await axios.post('http://194.195.119.206:9000/scan-document/', formData)
    scannedImage.value = `data:image/jpeg;base64,${response.data.image}`
  } catch (err) {
    error.value = err.response?.data?.detail || 'Failed to process document'
  } finally {
    loading.value = false
  }
}

const downloadScannedImage = () => {
  if (!scannedImage.value) return
  
  const link = document.createElement('a')
  link.href = scannedImage.value
  link.download = 'scanned_document.jpg'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}
</script>