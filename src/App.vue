<script setup>
import { ref, onMounted, nextTick } from 'vue';
import Papa from 'papaparse';
import { toBlob } from 'html-to-image';
import csvRaw from './assets/card_messages.csv?raw';

const showVideo = ref(false);
const showOverlay = ref(false);
const showShare = ref(false);
const isSharing = ref(false);
const videoRef = ref(null);
const wrapperRef = ref(null);

const currentTitle = ref("Titre de la carte");
const currentDesc = ref("description de la carte du jour");

onMounted(() => {
  const parsed = Papa.parse(csvRaw, {
    header: true,
    skipEmptyLines: true,
  });
  
  const rows = parsed.data;
  if (rows && rows.length > 0) {
    // Select a random row from the CSV
    const randomRow = rows[Math.floor(Math.random() * rows.length)];
    if (randomRow['Thème']) currentTitle.value = randomRow['Thème'];
    if (randomRow['Carte']) currentDesc.value = randomRow['Carte'];
  }
});

const playVideo = () => {
  showVideo.value = true;
};

const handleTimeUpdate = (event) => {
  if (event.target.currentTime >= 4.06 && !showOverlay.value) {
    showOverlay.value = true;
  }
};

const handleVideoEnded = () => {
  showShare.value = true;
};

const shareVisual = async () => {
  if (isSharing.value || !wrapperRef.value || !videoRef.value) return;
  
  isSharing.value = true;
  const oldShowShare = showShare.value;
  showShare.value = false;
  
  // Wait for the UI update so the share button disappears from the capture
  await nextTick();
  // Small delay to ensure rendering is complete
  await new Promise(r => setTimeout(r, 100));

  const wrapper = wrapperRef.value;
  const video = videoRef.value;
  
  // Create a canvas to draw the current video frame so html-to-image captures it perfectly
  const canvas = document.createElement('canvas');
  canvas.width = video.videoWidth || 1080;
  canvas.height = video.videoHeight || 1920;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
  
  // Position canvas directly over the video, under the overlay
  canvas.style.position = 'absolute';
  canvas.style.top = '0';
  canvas.style.left = '0';
  canvas.style.width = '100%';
  canvas.style.height = '100%';
  canvas.style.objectFit = 'cover';
  canvas.style.zIndex = '5'; // Video has default, overlay has 20, canvas on 5
  
  wrapper.appendChild(canvas);
  await new Promise(r => setTimeout(r, 50)); // let DOM update
  
  try {
    const blob = await toBlob(wrapper, { 
      pixelRatio: 2, 
      quality: 0.95,
      filter: (node) => {
        // Exclude the video node to heavily optimize things, since it's covered by the canvas
        return node.tagName !== 'VIDEO';
      }
    });

    if (!blob) throw new Error("Could not generate image blob");

    const file = new File([blob], 'effet-essor.png', { type: 'image/png' });
    
    if (navigator.canShare && navigator.canShare({ files: [file] })) {
      await navigator.share({
        title: 'Effet Essor',
        text: 'Ma réflexion quotidienne.',
        files: [file]
      });
    } else {
      // Fallback
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'effet-essor.png';
      a.click();
      URL.revokeObjectURL(url);
    }
  } catch (error) {
    console.error('Error sharing the image:', error);
    alert('Une erreur est survenue lors du partage.');
  } finally {
    wrapper.removeChild(canvas);
    showShare.value = oldShowShare;
    isSharing.value = false;
  }
};
</script>

<template>
  <main class="app-container">
    <header class="page-header">
      <h1 class="page-title">Effet Essor</h1>
      <p class="page-subtitle">Votre moment de réflexion quotidienne.</p>
    </header>

    <section class="content-section">
      <transition name="fade" mode="out-in">
        <!-- Hero Button State -->
        <div v-if="!showVideo" class="hero-card">
          <button @click="playVideo" class="primary-button">
            Voir la carte du jour
          </button>
        </div>
        
        <!-- Embedded Video State -->
        <div v-else class="video-container">
          <div class="video-wrapper" ref="wrapperRef">
            <!-- pointer-events-none added to CSS to prevent clicking to pause/play -->
            <video autoplay playsinline class="main-video" ref="videoRef" @timeupdate="handleTimeUpdate" @ended="handleVideoEnded">
              <source src="/effet-essor-video.mp4" type="video/mp4" />
              Votre navigateur ne supporte pas la balise vidéo.
            </video>
            <transition name="fade">
              <div v-if="showOverlay" class="card-overlay">
                <div class="overlay-top-section">
                  <h2 class="card-title">{{ currentTitle }}</h2>
                </div>
                <div class="overlay-bottom-section">
                  <p class="card-desc">{{ currentDesc }}</p>
                  <p class="card-author">@evelynehbrunet</p>
                </div>
              </div>
            </transition>
            
            <!-- Share Icon -->
            <transition name="fade">
              <button v-if="showShare" @click="shareVisual" class="share-button" :disabled="isSharing" aria-label="Partager">
                <svg v-if="!isSharing" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"></path>
                  <polyline points="16 6 12 2 8 6"></polyline>
                  <line x1="12" y1="2" x2="12" y2="15"></line>
                </svg>
                <svg v-else class="spinner" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <line x1="12" y1="2" x2="12" y2="6"></line>
                  <line x1="12" y1="18" x2="12" y2="22"></line>
                  <line x1="4.93" y1="4.93" x2="7.76" y2="7.76"></line>
                  <line x1="16.24" y1="16.24" x2="19.07" y2="19.07"></line>
                  <line x1="2" y1="12" x2="6" y2="12"></line>
                  <line x1="18" y1="12" x2="22" y2="12"></line>
                  <line x1="4.93" y1="19.07" x2="7.76" y2="16.24"></line>
                  <line x1="16.24" y1="7.76" x2="19.07" y2="4.93"></line>
                </svg>
              </button>
            </transition>
          </div>
        </div>
      </transition>
    </section>

    <footer class="page-footer">
      <!-- Future content can go here -->
    </footer>
  </main>
</template>

<style scoped>
/* App.vue specific styles here if needed. Rest is in main.css */
</style>
