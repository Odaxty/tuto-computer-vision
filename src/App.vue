<script setup>
import { ref, onMounted } from 'vue'
import '@tensorflow/tfjs-backend-cpu'
import '@tensorflow/tfjs-backend-webgl'
import * as cocoSsd from '@tensorflow-models/coco-ssd'

const videoRef = ref(null)
const canvasRef = ref(null)
const estEnChargement = ref(true)
let modeleIA = null

const demarrerTout = async () => {
  // Chargement du modèle
  modeleIA = await cocoSsd.load()
  estEnChargement.value = false

  // Accès caméra
  const stream = await navigator.mediaDevices.getUserMedia({
    video: {
      facingMode: 'user', // "environment" pour caméra arrière
      width: { ideal: 640 },
      height: { ideal: 480 }
    },
    audio: false
  })

  videoRef.value.srcObject = stream

  videoRef.value.onloadedmetadata = () => {
    videoRef.value.play()

    const w = videoRef.value.videoWidth
    const h = videoRef.value.videoHeight

    // Taille interne du canvas
    canvasRef.value.width = w
    canvasRef.value.height = h

    // Taille VISUELLE du canvas (CRUCIAL)
    canvasRef.value.style.width = w + 'px'
    canvasRef.value.style.height = h + 'px'

    detecterEnBoucle()
  }
}

const detecterEnBoucle = async () => {
  if (!videoRef.value || !canvasRef.value) return

  const predictions = await modeleIA.detect(videoRef.value)
  const ctx = canvasRef.value.getContext('2d')

  ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height)

  predictions.forEach(prediction => {
    const [x, y, width, height] = prediction.bbox
    const label = `${prediction.class} (${Math.round(prediction.score * 100)}%)`

    ctx.strokeStyle = '#00ffff'
    ctx.lineWidth = 3
    ctx.strokeRect(x, y, width, height)

    ctx.font = '16px Arial'
    ctx.fillStyle = '#00ffff'
    ctx.fillText(label, x, y - 8)
  })

  requestAnimationFrame(detecterEnBoucle)
}

onMounted(() => {
  demarrerTout()
})
</script>

<template>
  <main class="page">
    <h1 class="titre">
      {{ estEnChargement ? 'Chargement...' : 'I.A. Prête ! 👀' }}
    </h1>

    <div class="camera-container">
      <video ref="videoRef" muted playsinline></video>
      <canvas ref="canvasRef"></canvas>
    </div>

    <p class="info">
      Si c'est décalé, rafraîchis la page.
    </p>
  </main>
</template>

<style scoped>
.page {
  min-height: 100vh;
  background: #111;
  color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.titre {
  font-size: 2rem;
  margin-bottom: 1rem;
}

.camera-container {
  position: relative;
  width: fit-content;
  height: fit-content;
  border: 4px solid #00ffff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 0 25px rgba(0, 255, 255, 0.4);
}

video {
  display: block;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
}

.info {
  margin-top: 1rem;
  font-size: 0.9rem;
  color: #aaa;
}
</style>
