<script setup lang="ts">
import { ref, onMounted, onUnmounted, provide } from 'vue';

const screenshot = ref<string | null>(null);
const streamRef = ref<MediaStream | null>(null);
const videoRef = ref<HTMLVideoElement | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);
const magnifierRef = ref<HTMLDivElement | null>(null);
const zoom = 2;

const startStream = async () => {
  try {
    streamRef.value = await navigator.mediaDevices.getDisplayMedia();
  } catch (error) {
    console.error('Error starting display media:', error);
  }
};

const toggleScreenshot = async () => {
  if (screenshot.value) {
    screenshot.value = null;
    return;
  }

  if (!streamRef.value) return;

  if (!videoRef.value) {
    videoRef.value = document.createElement('video');
  }

  if (!canvasRef.value) {
    canvasRef.value = document.createElement('canvas');
  }

  const video = videoRef.value;
  video.srcObject = streamRef.value;

  try {
    await video.play();

    const canvas = canvasRef.value;
    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;

    const context = canvas.getContext('2d');
    if (context) {
      context.drawImage(video, 0, 0, canvas.width, canvas.height);
      screenshot.value = canvas.toDataURL('image/png');
    }
  } catch (error) {
    console.error('Error creating screenshot:', error);
  }
};

const moveMagnifier = (e: MouseEvent) => {
  if (!magnifierRef.value) return;

  magnifierRef.value.style.top = `${e.clientY - 50}px`;
  magnifierRef.value.style.left = `${e.clientX - 50}px`;
  magnifierRef.value.style.backgroundPosition = ``
}

onMounted(() => {
  startStream();
});

onUnmounted(() => {
  if (streamRef.value) {
    streamRef.value.getTracks().forEach((track) => track.stop());
    streamRef.value = null;
  }
});

provide('screenshot', screenshot);
provide('toggleScreenshot', toggleScreenshot);

</script>

<template>
  <div class="container">
    <slot />
    <template v-if="screenshot">
      <img
          :src="screenshot"
          alt="screenshot"
          class="screenshot"
          @mousemove="moveMagnifier"
      />
      <div ref="magnifierRef"
           class="magnifier"
           :style="{backgroundImage: `url(${screenshot})`, backgroundSize: `${1920 * zoom}px ${1080 * zoom}px`}"
      />
    </template>
  </div>
</template>

<style scoped>
.container {
  position: relative;
}

.screenshot {
  position: absolute;
  inset: 0;
  z-index: 1;
  opacity: 0;
}

.magnifier {
  position: absolute;
  z-index: 2;
  border: 3px solid #000;
  border-radius: 50%;
  cursor: none;
  width: 100px;
  height: 100px;
  top: 0;
  left: 0;
  pointer-events: none;
}
</style>