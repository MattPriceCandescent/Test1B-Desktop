<template>
  <aside class="fixed right-0 top-16 bottom-0 w-80 bg-white border-l border-gray-200 overflow-y-auto">
    <div class="p-6">
      <h2 class="text-lg font-bold text-gray-800 mb-6">Your progress</h2>

      <!-- Page-level Progress with nested form blocks -->
      <div class="relative">
        <!-- Connecting line -->
        <div class="absolute left-4 top-8 bottom-0 w-0.5 bg-gray-300"></div>
        
        <div
          v-for="(page, index) in pages"
          :key="page.id"
          class="relative mb-6"
        >
          <!-- Page indicator -->
          <div
            :class="[
              'flex items-center gap-3 mb-2',
              page.isActive ? 'text-blue-600 font-bold' : page.completionPercentage === 100 ? 'text-gray-600' : 'text-gray-400'
            ]"
          >
            <div
              :class="[
                'relative z-10 w-8 h-8 rounded-full flex items-center justify-center text-sm font-semibold flex-shrink-0',
                page.isActive 
                  ? 'bg-blue-600 text-white' 
                  : page.completionPercentage === 100 
                    ? 'bg-gray-200' 
                    : 'bg-gray-200 text-gray-500'
              ]"
            >
              <!-- Checkmark for 100% complete pages -->
              <svg 
                v-if="page.completionPercentage === 100" 
                :class="[
                  'w-5 h-5',
                  page.isActive ? 'text-white' : 'text-gray-600'
                ]" 
                fill="none" 
                stroke="currentColor" 
                viewBox="0 0 24 24"
              >
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
              </svg>
              <!-- Number for incomplete pages -->
              <span v-else>{{ index + 1 }}</span>
            </div>
            <button
              @click="handlePageClick(page.id)"
              :class="[
                'flex-1 text-left cursor-pointer hover:underline',
                page.isActive ? 'font-bold' : 'font-normal'
              ]"
            >
              {{ page.title }}
            </button>
          </div>

          <!-- Single progress bar per page, always visible -->
          <div class="ml-11 mt-2">
            <!-- When 100% complete: solid line -->
            <div 
              v-if="page.completionPercentage === 100"
              :class="[
                'w-full h-1 rounded-full',
                page.isActive ? 'bg-blue-600' : 'bg-gray-500'
              ]"
            ></div>
            <!-- When < 100%: progress bar with fill -->
            <div 
              v-else
              class="w-full bg-gray-200 rounded-full h-1"
            >
              <div
                :class="[
                  'h-1 rounded-full transition-all duration-300',
                  page.isActive ? 'bg-blue-600' : 'bg-gray-500'
                ]"
                :style="{ width: `${page.completionPercentage}%` }"
              ></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Floating Action Button -->
      <div class="fixed bottom-6 right-6">
        <button class="w-12 h-12 bg-blue-600 text-white rounded-full shadow-lg hover:bg-blue-700 transition-colors flex items-center justify-center">
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
          </svg>
        </button>
      </div>
    </div>
  </aside>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useOnboardingStore } from '../../stores/onboardingStore'
import { formData } from '../../data/formData'

const route = useRoute()
const router = useRouter()
const store = useOnboardingStore()

const currentPage = computed(() => {
  if (route.path.includes('/step1/page1')) return 'page1'
  if (route.path.includes('/step1/page2')) return 'page2'
  if (route.path.includes('/step1/page3')) return 'page3'
  return null
})

const pages = computed(() => {
  const step1Pages = formData.step1
  const pageIds = ['page1', 'page2', 'page3']
  
  return pageIds.map((pageId, index) => {
    const pageData = step1Pages[pageId]
    const isActive = currentPage.value === pageId
    const completionPercentage = store.calculatePageCompletionPercentage('step1', pageId)
    
    // Get form blocks for this page
    const blocks = pageData ? pageData.formBlocks.map(block => {
      const totalQuestions = block.questions.length
      const progress = store.calculateBlockProgress('step1', pageId, block.id, totalQuestions)
      return {
        id: block.id,
        title: block.title,
        progress
      }
    }) : []
    
    return {
      id: pageId,
      title: pageData ? pageData.title : '',
      isActive,
      completionPercentage,
      blocks
    }
  })
})

const handlePageClick = (pageId) => {
  // Progress is already saved in the store as user interacts with forms
  // Navigate to the clicked page
  const routeMap = {
    'page1': '/step1/page1',
    'page2': '/step1/page2',
    'page3': '/step1/page3'
  }
  
  const targetRoute = routeMap[pageId]
  if (targetRoute && route.path !== targetRoute) {
    router.push(targetRoute)
  }
}
</script>
