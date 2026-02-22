<script setup>
import { ref } from 'vue';

const showVideo = ref(false);
const showOverlay = ref(false);
const videoRef = ref(null);

const playVideo = () => {
  showVideo.value = true;
};

const handleTimeUpdate = (event) => {
  if (event.target.currentTime >= 4.06 && !showOverlay.value) {
    showOverlay.value = true;
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
          <div class="video-wrapper">
            <!-- pointer-events-none added to CSS to prevent clicking to pause/play -->
            <video autoplay playsinline class="main-video" ref="videoRef" @timeupdate="handleTimeUpdate">
              <source src="/effet-essor-video.mp4" type="video/mp4" />
              Votre navigateur ne supporte pas la balise vidéo.
            </video>
            <transition name="fade">
              <div v-if="showOverlay" class="card-overlay">
                <div class="overlay-top-section">
                  <h2 class="card-title">Titre de la carte</h2>
                </div>
                <div class="overlay-bottom-section">
                  <p class="card-desc">description de la carte du jour</p>
                </div>
              </div>
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
