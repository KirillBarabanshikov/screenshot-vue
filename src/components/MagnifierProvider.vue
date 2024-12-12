<script setup lang="ts">
  import { ref, onMounted, onUnmounted, provide } from 'vue';

  const screenshot = ref<string | null>(null);
  const streamRef = ref<MediaStream | null>(null);
  const videoRef = ref<HTMLVideoElement | null>(null);
  const canvasRef = ref<HTMLCanvasElement | null>(null);
  const magnifierRef = ref<HTMLDivElement | null>(null);
  const zoom = 2;
  const magnifierSize = 100;
  const screenWidth = (window as Window).innerWidth;
  const screenHeight = (window as Window).innerHeight;

  const startStream = async () => {
    try {
      streamRef.value = await navigator.mediaDevices.getDisplayMedia();
    } catch (error) {
      console.error('Error starting display media:', error);
    }
  };

  const toggleMagnifier = async () => {
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
    const magnifier = magnifierRef.value;
    let x = e.pageX;
    let y = e.pageY;
    const w = magnifier.clientWidth / 2;
    const h = magnifier.clientHeight / 2;
    const bw = 3;

    if (x > screenWidth - w / zoom) {
      x = screenWidth - w / zoom;
    }
    if (x < w / zoom) {
      x = w / zoom;
    }
    if (y > screenHeight - h / zoom) {
      y = screenHeight - h / zoom;
    }
    if (y < h / zoom) {
      y = h / zoom;
    }

    magnifier.style.left = x - w + 'px';
    magnifier.style.top = y - h + 'px';
    magnifier.style.backgroundPosition =
      '-' + (x * zoom - w + bw) + 'px -' + (y * zoom - h + bw) + 'px';
  };

  onMounted(() => {
    startStream();
  });

  onUnmounted(() => {
    if (streamRef.value) {
      streamRef.value.getTracks().forEach((track) => track.stop());
      streamRef.value = null;
    }
  });

  provide('toggleMagnifier', toggleMagnifier);
</script>

<template>
  <div class="container" @mousemove="moveMagnifier">
    <slot />
    <div
      v-if="screenshot"
      ref="magnifierRef"
      class="magnifier"
      :style="{
        width: `${magnifierSize}px`,
        height: `${magnifierSize}px`,
        backgroundImage: `url(${screenshot})`,
        backgroundSize: `${screenWidth * zoom}px ${screenHeight * zoom}px`,
      }"
    />
  </div>
</template>

<style scoped>
  .container {
    position: relative;
    overflow: hidden;
    width: 100vw;
    height: 100vh;
  }

  .magnifier {
    position: absolute;
    z-index: 2;
    border: 3px solid #000;
    border-radius: 50%;
    top: 0;
    left: 0;
    pointer-events: none;
    background-repeat: no-repeat;
  }
</style>
