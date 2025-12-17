<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import axios from 'axios'

// --- STATE MANAGEMENT ---
const cats = ref([]) 
const likedCats = ref([]) 
const currentIndex = ref(0) 
const isLoading = ref(true)
const isFinished = ref(false)
const showHistory = ref(false) 

// Dragging variables
const isDragging = ref(false)
const startX = ref(0)
const currentX = ref(0)
const cardElement = ref(null) 

// --- LOCAL STORAGE LOGIC ---
const loadLikes = () => {
  const saved = localStorage.getItem('my-cat-collection')
  if (saved) {
    likedCats.value = JSON.parse(saved)
  }
}

watch(likedCats, (newLikes) => {
  localStorage.setItem('my-cat-collection', JSON.stringify(newLikes))
}, { deep: true })

const clearHistory = () => {
  if (confirm('Are you sure you want to delete all your cats?')) {
    likedCats.value = []
  }
}

// --- OPTIMIZED API & DATA ---

// Helper: Preload a single image returning a Promise
const loadImage = (url) => {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.src = url
    img.onload = () => resolve(url)
    img.onerror = () => resolve(null) // Resolve even on error to keep app moving
  })
}

const fetchCats = async () => {
  isLoading.value = true
  try {
    // 1. Randomize ONLY the API call (to get new cats)
    const randomSkip = Math.floor(Math.random() * 2000)
    const response = await axios.get(`https://cataas.com/api/cats?limit=10&skip=${randomSkip}`)
    
    // 2. Process data
    const cleanData = response.data
      .filter(cat => cat._id || cat.id)
      .map(cat => {
        const realId = cat._id || cat.id
        return { 
          id: realId, 
          // FIX 1: REMOVED timestamp from image URL to allow Browser Caching!
          // We keep width=500 for performance.
          url: `https://cataas.com/cat/${realId}?width=500` 
        }
      })

    cats.value = cleanData
    
    // FIX 2: "Smart Wait"
    // We wait for the FIRST image to fully download before hiding the loading screen.
    if (cleanData.length > 0) {
      await loadImage(cleanData[0].url)
    }

    // Now that the first cat is ready, show the UI
    isLoading.value = false

    // Then secretly preload the rest in the background
    cleanData.slice(1).forEach(cat => {
      const img = new Image()
      img.src = cat.url
    })

  } catch (err) {
    console.error("API Failed:", err)
    alert("Cataas is slow right now. Please wait a moment.")
    isLoading.value = false
  }
}

const resetApp = () => {
  currentIndex.value = 0
  isFinished.value = false
  cats.value = [] 
  fetchCats() 
}

// --- INTERACTION HANDLERS ---
const handleStart = (e) => {
  if (showHistory.value) return 
  isDragging.value = true
  startX.value = e.type.includes('mouse') ? e.clientX : e.touches[0].clientX
}

const handleMove = (e) => {
  if (!isDragging.value) return
  const clientX = e.type.includes('mouse') ? e.clientX : e.touches[0].clientX
  currentX.value = clientX - startX.value
}

const handleEnd = () => {
  if (!isDragging.value) return
  isDragging.value = false
  
  const threshold = 100
  if (Math.abs(currentX.value) > threshold) {
    const direction = currentX.value < 0 ? 'left' : 'right'
    commitSwipe(direction)
  }
  currentX.value = 0
}

const commitSwipe = (direction) => {
  const currentCat = cats.value[currentIndex.value]
  
  if (direction === 'right') {
    const isDuplicate = likedCats.value.some(cat => cat.id === currentCat.id)
    if (!isDuplicate) {
      likedCats.value.push(currentCat)
    }
  }
  
  currentIndex.value++
  if (currentIndex.value >= cats.value.length) {
    isFinished.value = true
  }
}

const handleKeydown = (e) => {
  if (isFinished.value || isLoading.value || showHistory.value) return
  if (e.key === 'ArrowLeft') commitSwipe('left')
  if (e.key === 'ArrowRight') commitSwipe('right')
}

// --- CSS DYNAMICS ---
const cardStyle = computed(() => {
  if (!isDragging.value) {
    return { transition: 'transform 0.3s ease-out', transform: 'translate(0px, 0px)' }
  }
  const rotation = currentX.value * 0.05
  return {
    transition: 'none', 
    transform: `translate(${currentX.value}px, 0px) rotate(${rotation}deg)`,
    cursor: 'grabbing'
  }
})

onMounted(() => {
  loadLikes() 
  fetchCats()
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <div class="container">
    <header class="app-header">
      <h1>🐾 Meow Meow </h1>
      <div class="stats-badge" @click="showHistory = true">
        ❤️ {{ likedCats.length }}
      </div>
    </header>

    <div v-if="isLoading" class="status-msg">
      <div class="spinner"></div>
      <p>Summoning kitties...</p>
    </div>

    <div v-else-if="!isFinished" class="card-stack">
      <div 
        ref="cardElement"
        class="tinder-card"
        :style="cardStyle"
        @mousedown="handleStart"
        @touchstart="handleStart"
        @mousemove="handleMove"
        @touchmove="handleMove"
        @mouseup="handleEnd"
        @touchend="handleEnd"
        @mouseleave="handleEnd"
      >
        <img 
          :src="cats[currentIndex].url" 
          :key="cats[currentIndex].id"
          alt="Cat" 
          draggable="false" 
        />
        
        <div v-show="isDragging && currentX > 50" class="choice-stamp like">LIKE</div>
        <div v-show="isDragging && currentX < -50" class="choice-stamp nope">NOPE</div>

        <div class="instructions">Drag / Arrow Keys</div>
      </div>
      
      <div class="controls">
        <button @click="commitSwipe('left')" class="btn-dislike">❌</button>
        <button @click="showHistory = true" class="btn-history">📜</button>
        <button @click="commitSwipe('right')" class="btn-like">💚</button>
      </div>
    </div>

    <div v-else class="summary">
      <h2>Round Complete!</h2>
      <p>You have collected {{ likedCats.length }} cats total.</p>
      
      <div class="gallery">
        <div v-for="cat in likedCats.slice(-4)" :key="cat.id" class="gallery-item">
          <img :src="cat.url" loading="lazy" />
        </div>
      </div>
      
      <div class="summary-actions">
        <button @click="resetApp" class="btn-reset">Find More</button>
        <button @click="showHistory = true" class="btn-collection">Collection</button>
      </div>
    </div>

    <div v-if="showHistory" class="modal-overlay" @click.self="showHistory = false">
      <div class="modal-content">
        <div class="modal-header">
          <h3>Collection ({{ likedCats.length }})</h3>
          <button v-if="likedCats.length > 0" @click="clearHistory" class="btn-clear">Clear</button>
        </div>
        
        <div v-if="likedCats.length === 0" class="empty-state">
          No cats liked yet! 😿
        </div>

        <div class="mini-gallery">
          <div v-for="cat in likedCats.slice().reverse()" :key="cat.id" class="mini-item">
             <img :src="cat.url" />
          </div>
        </div>

        <button class="btn-close" @click="showHistory = false">Close</button>
      </div>
    </div>

  </div>
</template>

<style scoped>
/* --- LAYOUT --- */
.container {
  max-width: 400px; margin: 0 auto; text-align: center; padding: 20px; 
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  overflow: hidden; 
}
.app-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.app-header h1 { font-size: 1.5rem; margin: 0; }
.stats-badge {
  background: #ffecf0; color: #d94c64; padding: 8px 15px; border-radius: 20px;
  font-weight: bold; cursor: pointer; border: 1px solid #ffcbd4;
}

/* --- CARD --- */
.tinder-card {
  background: white; border-radius: 20px; box-shadow: 0 15px 30px rgba(0,0,0,0.1);
  overflow: hidden; position: relative; height: 60vh; display: flex; flex-direction: column; 
  border: 1px solid #eaeaea; cursor: grab; user-select: none; touch-action: none;
}
.tinder-card:active { cursor: grabbing; }
.tinder-card img { width: 100%; height: 100%; object-fit: cover; pointer-events: none; content-visibility: auto; will-change: transform; }

/* --- STAMPS --- */
.choice-stamp {
  position: absolute; top: 40px; padding: 5px 15px; border: 4px solid; border-radius: 10px;
  font-size: 2.5rem; font-weight: 800; text-transform: uppercase; z-index: 10;
  background: rgba(255, 255, 255, 0.2); pointer-events: none;
}
.like { color: #4cd964; border-color: #4cd964; right: 40px; transform: rotate(15deg); }
.nope { color: #ff3b30; border-color: #ff3b30; left: 40px; transform: rotate(-15deg); }

/* --- CONTROLS --- */
.controls { margin-top: 30px; display: flex; justify-content: center; gap: 20px; align-items: center; }
button {
  border: none; border-radius: 50%; cursor: pointer; transition: transform 0.2s;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1); display: flex; align-items: center; justify-content: center;
}
button:active { transform: scale(0.95); }
.btn-like, .btn-dislike { width: 70px; height: 70px; font-size: 1.8rem; background: white; }
.btn-like { color: #4cd964; border: 2px solid #4cd964; }
.btn-dislike { color: #ff3b30; border: 2px solid #ff3b30; }
.btn-history { width: 50px; height: 50px; font-size: 1.2rem; background: #f0f2f5; color: #333; }

/* --- NEW: SUMMARY BUTTONS --- */
.summary-actions {
  display: flex; justify-content: center; gap: 15px; margin-top: 30px;
}

/* Common style for rounded buttons */
.btn-reset, .btn-collection {
  width: auto; height: auto; padding: 12px 24px; border-radius: 50px; 
  font-size: 1rem; font-weight: bold; border: none; cursor: pointer; color: white;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
}

.btn-reset { background: #333; } /* Black/Dark Grey */
.btn-collection { background: #4a90e2; } /* Nice Blue */

/* --- MODAL --- */
.modal-overlay {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0,0,0,0.7); z-index: 100;
  display: flex; justify-content: center; align-items: center;
}
.modal-content {
  background: white; padding: 20px; border-radius: 20px;
  width: 85%; max-width: 350px; max-height: 80vh; overflow-y: auto;
}
.modal-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.mini-gallery { display: grid; grid-template-columns: repeat(3, 1fr); gap: 5px; margin: 20px 0; }
.mini-item img { width: 100%; aspect-ratio: 1; object-fit: cover; border-radius: 5px; }
.btn-close { width: 100%; border-radius: 10px; padding: 12px; background: #333; color: white; font-weight: bold; }
.btn-clear { background: none; border: none; color: #ff3b30; font-size: 0.9rem; padding: 5px 10px; box-shadow: none; border-radius: 5px; width: auto; height: auto;}
.btn-clear:hover { background: #ffebeb; }
.empty-state { padding: 40px 0; color: #888; }

/* --- UTILS --- */
.status-msg { margin-top: 50px; color: #666; }
.spinner {
  width: 40px; height: 40px; margin: 0 auto 20px;
  border: 4px solid #f3f3f3; border-top: 4px solid #333; border-radius: 50%;
  animation: spin 1s linear infinite;
}
@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
.gallery { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-top: 20px; }
.gallery-item img { width: 100%; border-radius: 8px; aspect-ratio: 1; object-fit: cover; }
.instructions {
  position: absolute; bottom: 20px; left: 0; right: 0; pointer-events: none;
  text-align: center; color: white; font-weight: bold; text-shadow: 0 2px 4px rgba(0,0,0,0.5);
}
</style>