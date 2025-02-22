<script setup>
import * as THREE from 'three'
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader'
import { onMounted, ref } from 'vue'

console.log('Three.js Version:', THREE.REVISION)

// Remove RouterLink and RouterView imports since we're not using them

let scene, camera, renderer, skull

// Add new state for intro animation
const showIntro = ref(true)

// Add progress state
const introProgress = ref(0)

// Add menu state and items
const isMenuOpen = ref(false)
const menuItems = [{ label: 'About' }, { label: 'Pricing' }, { label: 'Contact Us' }]

// Add menu timeout handling
let menuTimeout = null

// Add near the top of the script
const modelLoadError = ref(false)

function initThreeJS() {
  // Create scene
  scene = new THREE.Scene()

  // Create camera
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 35

  // Create renderer
  const container = document.querySelector('.skull-container')
  renderer = new THREE.WebGLRenderer({
    alpha: true,
    antialias: true, // Add antialiasing for smoother edges
  })
  renderer.setSize(800, 800)
  renderer.setClearColor(0x000000, 0)
  container.appendChild(renderer.domElement)

  // Update lights for glow effect
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
  const backLight = new THREE.PointLight(0x9f7aea, 2)
  backLight.position.set(0, 0, -10)
  const frontLight = new THREE.PointLight(0xffffff, 1)
  frontLight.position.set(0, 0, 10)
  scene.add(ambientLight, backLight, frontLight)

  // Create a basic skull shape instead of loading OBJ
  const geometry = new THREE.BoxGeometry(10, 12, 15)
  const material = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    emissive: 0x000000,
    specular: 0x666666,
    shininess: 10,
    transparent: true,
    opacity: 0.9,
  })

  skull = new THREE.Mesh(geometry, material)

  // Add wireframe
  const wireframe = new THREE.LineSegments(
    new THREE.WireframeGeometry(geometry),
    new THREE.LineBasicMaterial({
      color: 0xffffff,
      transparent: true,
      opacity: 0.2,
    }),
  )
  skull.add(wireframe)

  scene.add(skull)
  animate()
}

function animate() {
  requestAnimationFrame(animate)

  if (skull) {
    // Add subtle floating animation
    skull.position.y = Math.sin(Date.now() * 0.001) * 0.1
  }

  renderer.render(scene, camera)
}

// Adjust mouse tracking for smoother rotation
function handleMouseMove(event) {
  if (!skull) return

  // Calculate mouse position relative to the skull container
  const container = document.querySelector('.skull-container')
  const rect = container.getBoundingClientRect()
  const centerX = rect.left + rect.width / 2
  const centerY = rect.top + rect.height / 2

  // Calculate normalized offset from center
  const offsetX = (event.clientX - centerX) / (window.innerWidth / 2)
  const offsetY = (event.clientY - centerY) / (window.innerHeight / 2)

  // Calculate target rotations (inverted Y axis)
  const targetRotationY = (offsetX * Math.PI) / 4
  const targetRotationX = (offsetY * Math.PI) / 4 // Removed negative sign to invert

  // Apply smooth rotation
  skull.rotation.y += (targetRotationY - skull.rotation.y) * 0.1
  skull.rotation.x += (targetRotationX - skull.rotation.x) * 0.1
}

onMounted(() => {
  initThreeJS()
  startIntroSequence()

  // Add click outside listener
  document.addEventListener('click', handleClickOutside)

  // Add menu hover listeners
  const menuDropdown = document.querySelector('.menu-dropdown')
  menuDropdown?.addEventListener('mouseenter', clearMenuTimeout)
  menuDropdown?.addEventListener('mouseleave', startMenuTimeout)

  const sections = document.querySelectorAll('.content-section')
  sections.forEach((section) => {
    let currentBeam = null

    section.addEventListener('mouseenter', () => {
      if (currentBeam) currentBeam.remove()
      currentBeam = createBeam(section)
    })

    section.addEventListener('mouseleave', (event) => {
      // Check if we're not entering another content section
      const relatedTarget = event.relatedTarget
      if (!relatedTarget?.closest('.content-section')) {
        if (currentBeam) {
          currentBeam.classList.add('fade-out')
          setTimeout(() => currentBeam?.remove(), 300)
          currentBeam = null
        }
      }
    })
  })
})

function showContent() {
  const sections = document.querySelectorAll('.content-section')
  sections.forEach((section, index) => {
    setTimeout(() => {
      section.classList.add('show')
    }, index * 200) // Stagger the animations
  })
}

function createCircuits() {
  const container = document.querySelector('.circuit-container')
  const button = document.querySelector('.pulse-button')
  const skullContainer = document.querySelector('.skull-container')

  button.classList.add('fade-out')

  setTimeout(() => {
    button.style.display = 'none'
    skullContainer.classList.add('show')
    setTimeout(showContent, 500)
  }, 1000)

  const directions = [0, 45, 90, 135, 180, 225, 270, 315]

  directions.forEach((angle) => {
    const circuit = document.createElement('div')
    circuit.className = 'circuit'
    circuit.style.setProperty('--angle', `${angle}deg`)
    container.appendChild(circuit)

    for (let i = 0; i < 4; i++) {
      const branch = document.createElement('div')
      branch.className = 'circuit-branch'
      branch.style.setProperty('--delay', `${i * 0.1}s`)
      branch.style.setProperty('--distance', `${20 + i * 20}%`)
      branch.style.setProperty('--branch-angle', `${(Math.random() - 0.5) * 45}deg`)
      circuit.appendChild(branch)
    }

    setTimeout(() => {
      circuit.classList.add('fade-out')
      setTimeout(() => circuit.remove(), 200)
    }, 800)
  })
}

// Add beam tracking
let activeBeam = null

function createBeam(target) {
  const existingBeams = document.querySelectorAll('.skull-beam')
  existingBeams.forEach((beam) => beam.remove())

  const beam = document.createElement('div')
  beam.className = 'skull-beam'

  // Get skull position
  const skull = document.querySelector('.skull-container')
  const skullRect = skull.getBoundingClientRect()
  const skullCenter = {
    x: skullRect.left + skullRect.width / 2,
    y: skullRect.top + skullRect.height / 2,
  }

  // Get target position
  const targetRect = target.getBoundingClientRect()
  const targetCenter = {
    x: targetRect.left + targetRect.width / 2,
    y: targetRect.top + targetRect.height / 2,
  }

  // Calculate angle and distances
  const angle = Math.atan2(targetCenter.y - skullCenter.y, targetCenter.x - skullCenter.x)
  const skullRadius = 250
  const length = Math.hypot(targetCenter.x - skullCenter.x, targetCenter.y - skullCenter.y)

  // Calculate start position at skull edge
  const startX = skullCenter.x + Math.cos(angle) * skullRadius
  const startY = skullCenter.y + Math.sin(angle) * skullRadius

  // Set beam properties
  beam.style.setProperty('--beam-angle', `${angle}rad`)
  beam.style.setProperty('--beam-length', `${length - skullRadius}px`)
  beam.style.left = `${startX}px`
  beam.style.top = `${startY}px`

  // Add inner beam
  const innerBeam = document.createElement('div')
  innerBeam.className = 'skull-beam-inner'
  beam.appendChild(innerBeam)

  // Add to a fixed container instead of body
  const beamContainer = document.querySelector('.beam-container') || createBeamContainer()
  beamContainer.appendChild(beam)
  return beam
}

// Add a dedicated beam container
function createBeamContainer() {
  const container = document.createElement('div')
  container.className = 'beam-container'
  document.body.appendChild(container)
  return container
}

// Add intro animation sequence
function startIntroSequence() {
  const duration = 4000
  const startTime = Date.now()

  function updateProgress() {
    const elapsed = Date.now() - startTime
    introProgress.value = Math.min(elapsed / duration, 1)

    if (elapsed < duration) {
      requestAnimationFrame(updateProgress)
    } else {
      const introOverlay = document.querySelector('.intro-overlay')
      introOverlay.classList.add('fade-out')

      setTimeout(() => {
        showIntro.value = false
        const skullContainer = document.querySelector('.skull-container')
        skullContainer.classList.add('show')
        setTimeout(showContent, 1000)
      }, 800)
    }
  }

  updateProgress()
}

// Modified circuit creation for intro
function createAutoCircuits(directions) {
  const container = document.querySelector('.circuit-container')

  directions.forEach((angle) => {
    const circuit = document.createElement('div')
    circuit.className = 'circuit intro-circuit'
    circuit.style.setProperty('--angle', `${angle}deg`)
    container.appendChild(circuit)

    for (let i = 0; i < 4; i++) {
      const branch = document.createElement('div')
      branch.className = 'circuit-branch'
      branch.style.setProperty('--delay', `${i * 0.1}s`)
      branch.style.setProperty('--distance', `${20 + i * 20}%`)
      branch.style.setProperty('--branch-angle', `${(Math.random() - 0.5) * 45}deg`)
      circuit.appendChild(branch)
    }

    setTimeout(() => {
      circuit.classList.add('fade-out')
      setTimeout(() => circuit.remove(), 200)
    }, 800)
  })
}

function toggleMenu() {
  isMenuOpen.value = !isMenuOpen.value
  if (isMenuOpen.value) {
    startMenuTimeout()
  } else {
    clearMenuTimeout()
  }
}

function startMenuTimeout() {
  clearMenuTimeout()
  menuTimeout = setTimeout(() => {
    isMenuOpen.value = false
  }, 3000) // Close after 3 seconds of inactivity
}

function clearMenuTimeout() {
  if (menuTimeout) {
    clearTimeout(menuTimeout)
    menuTimeout = null
  }
}

// Add click outside handler
function handleClickOutside(event) {
  const header = document.querySelector('.site-header')
  if (isMenuOpen.value && header && !header.contains(event.target)) {
    isMenuOpen.value = false
  }
}
</script>

<template>
  <div class="hero" @mousemove="handleMouseMove">
    <div v-if="showIntro" class="intro-overlay">
      <div class="progress-arc top"></div>
      <div class="intro-text">
        <span class="welcome">Welcome to</span>
        <h1 class="brand">ImpactSync</h1>
      </div>
      <div class="progress-arc bottom"></div>
    </div>

    <!-- Update header section -->
    <header class="site-header" :class="{ show: !showIntro }">
      <div class="header-content" @click="toggleMenu" :class="{ 'menu-active': isMenuOpen }">
        <h1 class="brand">ImpactSync</h1>
        <div class="header-glow"></div>
        <div class="menu-icon">
          <span></span>
          <span></span>
          <span></span>
        </div>
      </div>
      <nav class="menu-dropdown" :class="{ show: isMenuOpen }">
        <div class="menu-item" v-for="item in menuItems" :key="item.label">
          <span class="menu-label">{{ item.label }}</span>
          <div class="menu-highlight"></div>
        </div>
      </nav>
    </header>

    <div class="circuit-container"></div>
    <div class="skull-container">
      <!-- Three.js skull renders here -->
    </div>

    <!-- Add content sections -->
    <div class="content-grid">
      <div class="content-section section-1">
        <h2>Create</h2>
        <p>Build something extraordinary</p>
      </div>

      <div class="content-section section-2">
        <h2>Innovate</h2>
        <p>Push the boundaries</p>
      </div>

      <div class="content-section section-3">
        <h2>Transform</h2>
        <p>Change the game</p>
      </div>

      <div class="content-section section-4">
        <h2>Evolve</h2>
        <p>Stay ahead of tomorrow</p>
      </div>
    </div>

    <!-- Add beam container -->
    <div class="beam-container"></div>
  </div>
</template>

<style>
/* Global styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --primary-purple: #6b46c1;
  --dark-purple: #44337a;
  --light-purple: #9f7aea;
}

html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
  background: #000;
}

.hero {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #000;
  z-index: 1;
  overflow: visible;
}

.pulse-button {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 1.5rem 3rem;
  font-size: 1.4rem;
  text-transform: uppercase;
  letter-spacing: 2px;
  background-color: var(--primary-purple);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  animation: pulse 2s ease-in-out infinite;
  z-index: 100;
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(107, 70, 193, 0.6);
  }
  70% {
    box-shadow: 0 0 0 20px rgba(107, 70, 193, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(107, 70, 193, 0);
  }
}

.pulse-button:hover {
  background-color: var(--dark-purple);
}

.pulse-button:active {
}

.circuit-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 90;
}

.circuit {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 100vmax; /* Ensures it reaches screen corners */
  height: 2px;
  transform-origin: left center;
  transform: rotate(var(--angle));
}

.circuit::after {
  content: '';
  position: absolute;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, var(--primary-purple), var(--light-purple));
  animation: spread 0.8s ease-out forwards;
  box-shadow:
    0 0 10px var(--light-purple),
    0 0 20px var(--primary-purple);
  opacity: 0;
}

.circuit-branch {
  position: absolute;
  left: var(--distance);
  width: 30%;
  height: 2px;
  background: linear-gradient(90deg, var(--primary-purple), transparent);
  transform-origin: left center;
  transform: rotate(var(--branch-angle));
  animation: branch 0.8s ease-out var(--delay) forwards;
  opacity: 0;
}

.fade-out .circuit::after,
.fade-out .circuit-branch {
  animation: fadeOut 0.5s ease-out forwards;
}

@keyframes spread {
  0% {
    opacity: 0;
    transform: scaleX(0);
  }
  10% {
    opacity: 1;
  }
  30%,
  70% {
    opacity: 0.8;
    transform: scaleX(1);
  }
  85% {
    opacity: 1;
  }
  100% {
    opacity: 0.8;
    transform: scaleX(1);
  }
}

@keyframes branch {
  0% {
    opacity: 0;
    transform: rotate(var(--branch-angle)) scaleX(0);
  }
  10% {
    opacity: 1;
  }
  30%,
  70% {
    opacity: 0.8;
    transform: rotate(var(--branch-angle)) scaleX(1);
  }
  85% {
    opacity: 1;
  }
  100% {
    opacity: 0.8;
    transform: rotate(var(--branch-angle)) scaleX(1);
  }
}

@keyframes fadeOut {
  from {
    opacity: 0.8;
  }
  to {
    opacity: 0;
  }
}

/* Add pulsing glow effect to the circuits */
.circuit::after,
.circuit-branch {
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

.pulse-button.fade-out {
  animation: buttonFadeOut 0.8s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}

@keyframes buttonFadeOut {
  0% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
  50% {
    opacity: 0.5;
    transform: translate(-50%, -50%) scale(1.2);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0);
  }
}

.skull-container {
  position: absolute;
  perspective: 1200px;
  transform: translate(-50%, -50%) scale(0.6);
  opacity: 0;
  transition: all 1.2s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 80;
  width: 800px;
  height: 800px;
  display: flex;
  justify-content: center;
  align-items: center;
  left: 50%;
  top: 50%;

  &::after {
    content: '';
    position: absolute;
    inset: -50%;
    background: radial-gradient(
      circle at center,
      rgba(159, 122, 234, 0.6) 0%,
      rgba(159, 122, 234, 0.4) 20%,
      rgba(159, 122, 234, 0.2) 40%,
      transparent 70%
    );
    filter: blur(60px);
    opacity: 0;
    transition: all 1.2s cubic-bezier(0.4, 0, 0.2, 1);
    z-index: -1;
    pointer-events: none;
  }
}

.skull-container.show {
  opacity: 1;
  transform: translate(-50%, -50%) scale(1);

  &::after {
    opacity: 1;
    filter: blur(40px);
    inset: -25%;
  }
}

.content-grid {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  pointer-events: none;
  z-index: 70;
}

.content-section {
  position: absolute;
  padding: 2rem;
  width: 300px; /* Add fixed width */
  background: rgba(107, 70, 193, 0.1);
  border: 1px solid rgba(159, 122, 234, 0.3);
  border-radius: 12px;
  backdrop-filter: blur(8px);
  opacity: 0;
  transform: translateY(20px) rotate(var(--rotate-angle)); /* Combine transforms */
  transition: all 0.3s ease;
  pointer-events: auto;
  cursor: pointer;
  z-index: 90; /* Increased to be above beams */
}

.content-section.show {
  opacity: 1;
  transform: translateY(0) rotate(var(--rotate-angle));
}

.content-section h2 {
  color: var(--light-purple);
  font-size: 2rem;
  margin-bottom: 0.5rem;
}

.content-section p {
  color: white;
  opacity: 0.8;
}

/* Update section positions */
.section-1 {
  top: 15%;
  left: 15%;
  --rotate-angle: -5deg;
}

.section-2 {
  top: 15%;
  right: 15%;
  --rotate-angle: 3deg;
}

.section-3 {
  bottom: 15%;
  left: 15%;
  --rotate-angle: 2deg;
}

.section-4 {
  bottom: 15%;
  right: 15%;
  --rotate-angle: -3deg;
}

/* Update hover effect */
.content-section:hover {
  transform: scale(1.05) rotate(0deg);
  background: rgba(107, 70, 193, 0.3);
  border-color: var(--light-purple);
  box-shadow:
    0 0 40px rgba(159, 122, 234, 0.4),
    0 0 80px rgba(159, 122, 234, 0.2);
  transition: all 0.3s ease;
}

/* Update beam styles for better blending */
.skull-beam {
  position: absolute; /* Changed from fixed since container is fixed */
  width: var(--beam-length);
  height: 40px;
  background: linear-gradient(
    90deg,
    rgba(159, 122, 234, 0.8) -20%,
    /* Increased opacity and adjusted start */ rgba(159, 122, 234, 0.6) 10%,
    rgba(159, 122, 234, 0.3) 50%,
    transparent 100%
  );
  transform-origin: left center;
  transform: rotate(var(--beam-angle));
  filter: blur(20px);
  pointer-events: none;
  z-index: 85;
  animation: beamIn 0.4s ease-out forwards;
  box-shadow:
    0 0 60px rgba(159, 122, 234, 0.4),
    0 0 120px rgba(159, 122, 234, 0.2);
  mix-blend-mode: screen;
  opacity: 0.9; /* Increased from 0.8 */
}

.skull-beam-inner {
  position: absolute;
  width: 100%;
  height: 8px;
  top: 50%;
  transform: translateY(-50%);
  background: linear-gradient(
    90deg,
    rgba(159, 122, 234, 1) -10%,
    rgba(159, 122, 234, 0.8) 20%,
    rgba(159, 122, 234, 0.4) 60%,
    transparent 100%
  );
  filter: blur(4px);
  animation: beamPulse 2s ease-in-out infinite;
}

@keyframes beamPulse {
  0%,
  100% {
    opacity: 0.8;
    filter: blur(4px);
    transform: translateY(-50%) scaleY(1);
  }
  50% {
    opacity: 1;
    filter: blur(2px);
    transform: translateY(-50%) scaleY(1.2);
  }
}

@keyframes beamIn {
  from {
    opacity: 0;
    transform: rotate(var(--beam-angle)) scaleX(0.5);
  }
  to {
    opacity: 1;
    transform: rotate(var(--beam-angle)) scaleX(1);
  }
}

.skull-beam.fade-out {
  animation: beamOut 0.3s ease-in forwards;
}

@keyframes beamOut {
  from {
    opacity: 1;
    transform: rotate(var(--beam-angle)) scaleX(1);
  }
  to {
    opacity: 0;
    transform: rotate(var(--beam-angle)) scaleX(0.8);
  }
}

/* Add pulsing effect to content sections when beam is active */
.content-section:hover {
  animation: contentPulse 2s ease-in-out infinite;
}

@keyframes contentPulse {
  0%,
  100% {
    box-shadow:
      0 0 30px rgba(159, 122, 234, 0.2),
      0 0 60px rgba(159, 122, 234, 0.1);
  }
  50% {
    box-shadow:
      0 0 40px rgba(159, 122, 234, 0.4),
      0 0 80px rgba(159, 122, 234, 0.2);
  }
}

/* Update intro styles */
.intro-overlay {
  position: fixed;
  inset: 0;
  background: radial-gradient(circle at center, #000 0%, #000 70%, rgba(159, 122, 234, 0.1) 100%);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

.intro-overlay.fade-out {
  opacity: 0;
  background: transparent;
}

.progress-arc {
  position: absolute;
  width: 400px; /* Increased width */
  height: 200px; /* Increased height */
  border: 2px solid var(--light-purple);
  opacity: 0.5;
  transform-origin: center;
  filter: drop-shadow(0 0 20px rgba(159, 122, 234, 0.2));
}

.progress-arc.top {
  border-radius: 200px 200px 0 0;
  border-bottom: none;
  top: calc(50% - 280px); /* Increased distance */
  animation: arcProgress 4s ease-in-out forwards;
}

.progress-arc.bottom {
  border-radius: 0 0 200px 200px;
  border-top: none;
  bottom: calc(50% - 280px); /* Increased distance */
  animation: arcProgress 4s ease-in-out forwards;
}

@keyframes arcProgress {
  0% {
    transform: scale(1);
    opacity: 0;
  }
  20% {
    transform: scale(1);
    opacity: 0.5;
  }
  100% {
    transform: scale(0.8);
    opacity: 0;
  }
}

/* Update header styles for a more modern look */
.site-header {
  position: fixed;
  top: 2rem;
  left: 50%;
  transform: translateX(-50%) translateY(-100%);
  opacity: 0;
  transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 95;
}

.header-content {
  position: relative;
  padding: 0.75rem 2rem;
  background: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(10px);
  border-radius: 100px;
  border: 1px solid rgba(159, 122, 234, 0.2);
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 1rem;
}

.header-glow {
  position: absolute;
  inset: -1px;
  border-radius: 100px;
  background: linear-gradient(90deg, transparent, rgba(159, 122, 234, 0.2), transparent);
  opacity: 0;
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.site-header.show {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}

.site-header:hover .header-content {
  transform: scale(1.05);
  background: rgba(0, 0, 0, 0.5);
  border-color: rgba(159, 122, 234, 0.4);
}

.site-header:hover .header-glow {
  opacity: 1;
  filter: blur(8px);
}

.site-header .brand {
  font-size: 1.75rem;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  font-weight: 500;
  background: linear-gradient(90deg, #fff, var(--light-purple));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 0 10px rgba(159, 122, 234, 0.3));
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.site-header:hover .brand {
  filter: drop-shadow(0 0 15px rgba(159, 122, 234, 0.5));
  letter-spacing: 0.06em;
}

/* Update brand styles */
.brand {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 5rem;
  font-weight: 700;
  color: white;
  letter-spacing: -0.02em;
  margin: 0;
  background: linear-gradient(120deg, #fff, var(--light-purple));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 0 30px rgba(159, 122, 234, 0.3));
  animation: textGlow 2s ease-in-out infinite;
}

.site-header .brand {
  font-size: 3rem;
  animation: none;
  transition: transform 0.3s ease;
}

.site-header:hover .brand {
  transform: scale(1.05);
}

@keyframes textGlow {
  0%,
  100% {
    filter: drop-shadow(0 0 30px rgba(159, 122, 234, 0.3));
    transform: scale(1);
  }
  50% {
    filter: drop-shadow(0 0 50px rgba(159, 122, 234, 0.5));
    transform: scale(1.02);
  }
}

/* Add beam container styles */
.beam-container {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 85;
}

/* Add menu icon styles */
.menu-icon {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding-left: 1rem;
  margin-left: 1rem;
  border-left: 1px solid rgba(159, 122, 234, 0.2);
}

.menu-icon span {
  display: block;
  width: 20px;
  height: 2px;
  background: var(--light-purple);
  transition: all 0.3s ease;
}

.menu-active .menu-icon span:nth-child(1) {
  transform: translateY(6px) rotate(45deg);
}

.menu-active .menu-icon span:nth-child(2) {
  opacity: 0;
}

.menu-active .menu-icon span:nth-child(3) {
  transform: translateY(-6px) rotate(-45deg);
}

/* Add dropdown menu styles */
.menu-dropdown {
  position: absolute;
  top: calc(100% + 1rem);
  left: 50%;
  transform: translateX(-50%) translateY(-10px);
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  border: 1px solid rgba(159, 122, 234, 0.2);
  padding: 0.5rem;
  min-width: 200px;
  opacity: 0;
  visibility: hidden;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.menu-dropdown.show {
  opacity: 1;
  visibility: visible;
  transform: translateX(-50%) translateY(0);
  animation: slideDown 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

.menu-item {
  position: relative;
  padding: 1rem 1.5rem;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  overflow: hidden;
}

.menu-label {
  position: relative;
  z-index: 1;
}

.menu-highlight {
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, var(--primary-purple), transparent);
  opacity: 0;
  transition: all 0.3s ease;
}

.menu-item:hover .menu-highlight {
  opacity: 1;
  transform: translateX(0);
}

.menu-item:hover {
  color: var(--light-purple);
}

/* Update header hover states */
.header-content.menu-active {
  border-color: var(--light-purple);
  background: rgba(0, 0, 0, 0.6);
  border-radius: 24px;
}
</style>
