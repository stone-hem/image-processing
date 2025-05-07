<template>
  <div>
    <h2 class="text-xl font-semibold mb-6 text-gray-800">OCR Text Extraction</h2>
    
    <div 
      @click="triggerFileInput"
      class="border-2 border-dashed border-gray-300 rounded-lg p-8 text-center cursor-pointer hover:border-green-500 transition-colors mb-6"
    >
      <p class="text-gray-500">Click to upload an image for text extraction</p>
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
    
    <div v-if="imagePreview" class="mb-6">
      <h3 class="font-medium text-gray-700 mb-2">Uploaded Image</h3>
      <img :src="imagePreview" class="max-w-full h-auto rounded border border-gray-200 max-h-80">
    </div>
    
    <div v-if="extractedText" class="bg-gray-50 rounded-lg p-4">
      <h3 class="font-medium text-gray-700 mb-2">Extracted Text</h3>
      <textarea 
        v-model="extractedText" 
        readonly
        class="w-full h-64 p-3 border border-gray-300 rounded-md bg-white"
      ></textarea>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const fileInput = ref(null)
const imagePreview = ref(null)
const extractedText = ref(null)
const loading = ref(false)
const error = ref(null)

const triggerFileInput = () => fileInput.value.click()

const handleFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  try {
    loading.value = true
    error.value = null
    extractedText.value = null
    
    imagePreview.value = URL.createObjectURL(file)
    
    const formData = new FormData()
    formData.append('file', file)
    
    const response = await axios.post('http://194.195.119.206:9000/extract-text/', formData)
    extractedText.value = response.data.text
  } catch (err) {
    error.value = err.response?.data?.detail || 'Failed to extract text'
  } finally {
    loading.value = false
  }
}
</script>