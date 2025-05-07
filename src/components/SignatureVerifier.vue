<template>
  <div>
    <h2 class="text-xl font-semibold mb-6 text-gray-800">Signature Verification</h2>
    
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
      <div 
        @click="triggerFileInput('document')"
        class="border-2 border-dashed border-gray-300 rounded-lg p-4 text-center cursor-pointer hover:border-green-500 transition-colors h-full"
      >
        <p class="text-gray-500">Upload Document</p>
        <input 
          type="file" 
          ref="documentInput" 
          @change="handleDocumentUpload" 
          accept="image/*" 
          class="hidden"
        >
      </div>
      
      <div 
        @click="triggerFileInput('signature')"
        class="border-2 border-dashed border-gray-300 rounded-lg p-4 text-center cursor-pointer hover:border-green-500 transition-colors h-full"
      >
        <p class="text-gray-500">Upload Signature</p>
        <input 
          type="file" 
          ref="signatureInput" 
          @change="handleSignatureUpload" 
          accept="image/*" 
          class="hidden"
        >
      </div>
      
      <div 
        @click="triggerFileInput('reference')"
        class="border-2 border-dashed border-gray-300 rounded-lg p-4 text-center cursor-pointer hover:border-green-500 transition-colors h-full"
      >
        <p class="text-gray-500">Upload Reference Signature (Optional)</p>
        <input 
          type="file" 
          ref="referenceInput" 
          @change="handleReferenceUpload" 
          accept="image/*" 
          class="hidden"
        >
      </div>
    </div>
    
    <div v-if="loading" class="text-center py-4">
      <div class="inline-block animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-green-600"></div>
      <p class="mt-2 text-gray-600">Processing...</p>
    </div>
    
    <div v-if="error" class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 mb-6">
      <p>{{ error }}</p>
    </div>
    
    <div v-if="documentPreview || signaturePreview || referencePreview" class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
      <div v-if="documentPreview" class="bg-gray-50 p-4 rounded-lg">
        <h3 class="font-medium text-gray-700 mb-2">Document</h3>
        <img :src="documentPreview" class="max-w-full h-auto rounded border border-gray-200">
      </div>
      <div v-if="signaturePreview" class="bg-gray-50 p-4 rounded-lg">
        <h3 class="font-medium text-gray-700 mb-2">Signature</h3>
        <img :src="signaturePreview" class="max-w-full h-auto rounded border border-gray-200">
      </div>
      <div v-if="referencePreview" class="bg-gray-50 p-4 rounded-lg">
        <h3 class="font-medium text-gray-700 mb-2">Reference Signature</h3>
        <img :src="referencePreview" class="max-w-full h-auto rounded border border-gray-200">
      </div>
    </div>
    
    <button 
      @click="verifySignature" 
      :disabled="!documentPreview || !signaturePreview || loading"
      class="w-full md:w-auto bg-green-600 hover:bg-green-700 text-white px-6 py-3 rounded-md transition-colors disabled:bg-gray-400 disabled:cursor-not-allowed"
    >
      Verify Signature
    </button>
    
    <div v-if="verificationResult" class="mt-6 bg-gray-50 rounded-lg p-4">
      <h3 class="font-medium text-gray-700 mb-2">Verification Result</h3>
      <div class="flex items-center gap-2">
        <span>Status:</span>
        <span 
          class="font-semibold"
          :class="verificationResult.is_verified ? 'text-green-600' : 'text-red-600'"
        >
          {{ verificationResult.is_verified ? 'Verified' : 'Not Verified' }}
        </span>
      </div>
      <div class="flex items-center gap-2 mt-1">
        <span>Confidence:</span>
        <span class="font-medium">{{ (verificationResult.confidence * 100).toFixed(2) }}%</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const documentInput = ref(null)
const signatureInput = ref(null)
const referenceInput = ref(null)

const documentPreview = ref(null)
const signaturePreview = ref(null)
const referencePreview = ref(null)

const verificationResult = ref(null)
const loading = ref(false)
const error = ref(null)

const triggerFileInput = (type) => {
  if (type === 'document') documentInput.value.click()
  else if (type === 'signature') signatureInput.value.click()
  else if (type === 'reference') referenceInput.value.click()
}

const handleDocumentUpload = (event) => {
  const file = event.target.files[0]
  if (file) documentPreview.value = URL.createObjectURL(file)
}

const handleSignatureUpload = (event) => {
  const file = event.target.files[0]
  if (file) signaturePreview.value = URL.createObjectURL(file)
}

const handleReferenceUpload = (event) => {
  const file = event.target.files[0]
  if (file) referencePreview.value = URL.createObjectURL(file)
}

const verifySignature = async () => {
  if (!documentPreview.value || !signaturePreview.value) return
  
  try {
    loading.value = true
    error.value = null
    verificationResult.value = null
    
    const formData = new FormData()
    formData.append('document', documentInput.value.files[0])
    formData.append('signature', signatureInput.value.files[0])
    
    if (referenceInput.value?.files[0]) {
      formData.append('reference_signature', referenceInput.value.files[0])
    }
    
    const response = await axios.post('http://194.195.119.206:9000/verify-signature/', formData)
    verificationResult.value = response.data
  } catch (err) {
    error.value = err.response?.data?.detail || 'Failed to verify signature'
  } finally {
    loading.value = false
  }
}
</script>