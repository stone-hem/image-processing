<template>
  <div class="min-h-screen bg-gray-50 flex flex-col">
    <header class="bg-green-600 text-white shadow-md">
      <div class="container mx-auto px-4 py-6">
        <h1 class="text-3xl font-bold text-center">Document Scanner with OCR & Signature Verification</h1>
      </div>
    </header>
    
    <nav class="bg-white shadow-sm">
      <div class="container mx-auto px-4 flex justify-center space-x-2 md:space-x-6 py-4">
        <button 
          v-for="tab in tabs"
          :key="tab.id"
          @click="activeTab = tab.id"
          class="px-4 py-2 rounded-md transition-colors text-sm md:text-base"
          :class="activeTab === tab.id 
            ? 'bg-green-600 text-white shadow-md' 
            : 'bg-gray-100 hover:bg-gray-200 text-gray-800'"
        >
          {{ tab.label }}
        </button>
      </div>
    </nav>
    
    <main class="flex-grow container mx-auto px-4 py-8">
      <div class="bg-white rounded-lg shadow-xl overflow-hidden">
        <div v-if="activeTab === 'scanner'" class="p-6">
          <DocumentScanner />
        </div>
        <div v-if="activeTab === 'ocr'" class="p-6">
          <OCRViewer />
        </div>
        <div v-if="activeTab === 'signature'" class="p-6">
          <SignatureVerifier />
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import DocumentScanner from './components/DocumentScanner.vue'
import OCRViewer from './components/OCRViewer.vue'
import SignatureVerifier from './components/SignatureVerifier.vue'

const tabs = [
  { id: 'scanner', label: 'Document Scanner' },
  { id: 'ocr', label: 'OCR Text Extraction' },
  { id: 'signature', label: 'Signature Verification' }
]

const activeTab = ref('scanner')
</script>