<script setup>
import * as THREE from 'three'
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader'
import { onMounted, ref, onUnmounted } from 'vue'
import { ShaderMaterial } from 'three'

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

// Update device detection to be reactive
const isMobile = ref(window.innerWidth <= 768)

// Add content sections data
const sections = [
  { title: 'Create', description: 'Build something extraordinary' },
  { title: 'Innovate', description: 'Push the boundaries' },
  { title: 'Transform', description: 'Change the game' },
  { title: 'Evolve', description: 'Stay ahead of tomorrow' },
]

// Add a reference to track the container
let currentContainer = null

// Update the color variables at the top
const COLORS = {
  primary: '#0088ff', // Electric blue
  dark: '#005299', // Darker blue
  light: '#40a9ff', // Light blue
  glow: '#00e5ff', // Cyan glow
}

// Update the shader code with more prominent swirls
const energyShader = {
  vertexShader: `
    varying vec2 vUv;
    varying vec3 vPosition;
    void main() {
      vUv = uv;
      vPosition = position;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    uniform float time;
    varying vec2 vUv;
    varying vec3 vPosition;

    float noise(vec3 p) {
      return sin(p.x * 15.0 + time) * // Increased frequency from 10.0 to 15.0
             cos(p.y * 12.0 - time * 0.5) * // Increased frequency and speed
             sin(p.z * 18.0 + time * 0.7); // Increased frequency and variation
    }

    void main() {
      vec3 color = vec3(0.0, 0.53, 1.0);

      // Create more intense swirling layers
      float noise1 = noise(vPosition * 0.8); // Increased scale
      float noise2 = noise(vPosition * 1.2 + vec3(time * 0.3)); // Faster movement
      float noise3 = noise(vPosition * 1.6 - vec3(time * 0.4)); // More variation

      // Combine noise layers with higher contrast
      float finalNoise = (noise1 + noise2 + noise3) / 2.5; // Reduced division for more intensity

      // Enhanced "thinking" pulses
      float pulse1 = sin(time * 1.2) * 0.5 + 0.5; // Faster pulses
      float pulse2 = sin(time * 1.8 + 2.0) * 0.5 + 0.5;
      float pulse3 = sin(time * 0.9 + 4.0) * 0.5 + 0.5;

      // More prominent synapses
      float synapses = step(0.88, sin(vPosition.x * 25.0 + time) * // Increased frequency and lowered threshold
                                sin(vPosition.y * 25.0 - time) *
                                sin(vPosition.z * 25.0 + time * 0.8));

      // Combine effects with higher intensity
      float alpha = (finalNoise * 0.6 + 0.4) * // Increased base opacity
                   mix(0.3, 0.6, (pulse1 + pulse2 + pulse3) / 3.0) +
                   synapses * 0.7; // More prominent flashes

      // Enhanced color variation
      color = mix(color, vec3(0.2, 0.8, 1.0), finalNoise * 0.7); // More color contrast
      color += vec3(0.2, 0.4, 0.8) * synapses; // Brighter flashes

      gl_FragColor = vec4(color, alpha);
    }
  `,
}

function initThreeJS() {
  // Clean up existing scene and renderer
  cleanupThreeJS()

  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, 1, 0.1, 1000)
  camera.position.z = 60

  const container = document.querySelector('.skull-container')
  if (!container) return

  currentContainer = container

  renderer = new THREE.WebGLRenderer({
    alpha: true,
    antialias: true,
  })

  const size = Math.min(container.clientWidth, container.clientHeight)
  renderer.setSize(size, size)
  renderer.setClearColor(0x000000, 0)
  container.appendChild(renderer.domElement)

  // Create larger complex sphere geometry with enhanced materials
  const geometry = new THREE.IcosahedronGeometry(30, 2)
  const material = new THREE.MeshPhongMaterial({
    color: 0x0088ff,
    wireframe: true,
    transparent: true,
    opacity: 0.2,
    wireframeLinewidth: 0.8,
  })

  skull = new THREE.Mesh(geometry, material)

  // Add inner sphere with glowing effect
  const innerGeometry = new THREE.IcosahedronGeometry(28, 1)
  const innerMaterial = new THREE.MeshPhongMaterial({
    color: 0x00e5ff,
    wireframe: true,
    transparent: true,
    opacity: 0.15,
    wireframeLinewidth: 0.8,
  })
  const innerSphere = new THREE.Mesh(innerGeometry, innerMaterial)
  skull.add(innerSphere)

  // Add outer glow sphere
  const outerGeometry = new THREE.IcosahedronGeometry(32, 1)
  const outerMaterial = new THREE.MeshPhongMaterial({
    color: 0x40a9ff,
    wireframe: true,
    transparent: true,
    opacity: 0.1,
    wireframeLinewidth: 0.8,
  })
  const outerSphere = new THREE.Mesh(outerGeometry, outerMaterial)
  skull.add(outerSphere)

  // Add particle system for enhanced effect
  const particlesGeometry = new THREE.BufferGeometry()
  const particleCount = 200
  const positions = new Float32Array(particleCount * 3)

  for (let i = 0; i < particleCount * 3; i += 3) {
    const radius = 35
    const theta = Math.random() * Math.PI * 2
    const phi = Math.acos(Math.random() * 2 - 1)

    positions[i] = radius * Math.sin(phi) * Math.cos(theta)
    positions[i + 1] = radius * Math.sin(phi) * Math.sin(theta)
    positions[i + 2] = radius * Math.cos(phi)
  }

  particlesGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

  const particlesMaterial = new THREE.PointsMaterial({
    color: 0x00e5ff,
    size: 0.5,
    transparent: true,
    opacity: 0.6,
    blending: THREE.AdditiveBlending,
  })

  const particles = new THREE.Points(particlesGeometry, particlesMaterial)
  skull.add(particles)

  // Create energy sphere
  const energyGeometry = new THREE.IcosahedronGeometry(25, 2)
  const energyMaterial = new ShaderMaterial({
    vertexShader: energyShader.vertexShader,
    fragmentShader: energyShader.fragmentShader,
    uniforms: {
      time: { value: 0 },
    },
    transparent: true,
    blending: THREE.AdditiveBlending,
  })
  const energySphere = new THREE.Mesh(energyGeometry, energyMaterial)
  skull.add(energySphere)

  scene.add(skull)

  // Enhanced lighting
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
  scene.add(ambientLight)

  const pointLight1 = new THREE.PointLight(0x00e5ff, 2)
  pointLight1.position.set(20, 20, 20)
  scene.add(pointLight1)

  const pointLight2 = new THREE.PointLight(0x0088ff, 1)
  pointLight2.position.set(-20, -20, -20)
  scene.add(pointLight2)

  // Add volumetric light effect
  const volumetricLight = new THREE.SpotLight(0x00e5ff, 1)
  volumetricLight.position.set(0, 30, 0)
  volumetricLight.angle = Math.PI / 4
  volumetricLight.penumbra = 0.5
  scene.add(volumetricLight)

  animate()
}

// Update the animation function to make the movement more dynamic
function animate() {
  if (!renderer || !scene || !camera || !skull) return
  requestAnimationFrame(animate)

  // Update energy shader time with more dynamic variations
  const energyMesh = skull.children[3]
  if (energyMesh && energyMesh.material.uniforms) {
    const baseSpeed = 0.015 // Increased base speed
    const burstSpeed = Math.sin(Date.now() * 0.002) * 0.02 + 0.025 // More pronounced bursts
    energyMesh.material.uniforms.time.value += baseSpeed + burstSpeed

    // More pronounced rotation variations
    energyMesh.rotation.x += Math.sin(Date.now() * 0.001) * 0.002
    energyMesh.rotation.y += Math.cos(Date.now() * 0.0007) * 0.002
  }

  // Rotate particles in opposite direction for enhanced effect
  if (skull.children[4]) {
    // particles are now the fifth child
    skull.children[4].rotation.y -= 0.001
    skull.children[4].rotation.x -= 0.001
  }

  renderer.render(scene, camera)
}

// Update cleanup function
function cleanupThreeJS() {
  if (skull) {
    scene?.remove(skull)
    skull = null
  }

  if (scene) {
    while (scene.children.length > 0) {
      scene.remove(scene.children[0])
    }
  }

  if (renderer) {
    renderer.dispose()
  }

  if (currentContainer) {
    while (currentContainer.firstChild) {
      currentContainer.removeChild(currentContainer.firstChild)
    }
  }

  scene = null
  renderer = null
  currentContainer = null
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
  handleResize() // Check mobile state first
  if (!isMobile.value) {
    initThreeJS()
  }
  startIntroSequence()

  // Add click outside listener
  document.addEventListener('click', handleClickOutside)

  // Add menu hover listeners
  const menuDropdown = document.querySelector('.menu-dropdown')
  menuDropdown?.addEventListener('mouseenter', clearMenuTimeout)
  menuDropdown?.addEventListener('mouseleave', startMenuTimeout)

  document.addEventListener('touchstart', handleTouchStart)
  document.addEventListener('touchmove', (e) => handleTouchStart(e))
})

onUnmounted(() => {
  cleanupThreeJS()
  window.removeEventListener('resize', handleResize)
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

// Add touch event handling
function handleTouchStart(event) {
  if (!skull || isMobile.value) return
  handleMouseMove(event.touches[0])
}

// Update window resize handling
function handleResize() {
  isMobile.value = window.innerWidth <= 768
  if (!isMobile.value && !skull) {
    initThreeJS()
  }
}

// Update mouse tracking function
function handleMouseMove(event) {
  if (!skull || isMobile.value) return

  // Use viewport dimensions
  const viewportWidth = window.innerWidth
  const viewportHeight = window.innerHeight
  const centerX = viewportWidth / 2
  const centerY = viewportHeight / 2

  // Calculate normalized position (-1 to 1)
  const x = (event.clientX - centerX) / (viewportWidth / 2)
  const y = (event.clientY - centerY) / (viewportHeight / 2)

  // Calculate rotation with more pronounced effect
  const targetRotationX = y * (Math.PI / 4) // 45 degrees max, inverted Y axis
  const targetRotationY = x * (Math.PI / 4) // 45 degrees max

  // Apply rotation with smoother easing
  requestAnimationFrame(() => {
    if (skull) {
      skull.rotation.x += (targetRotationX - skull.rotation.x) * 0.15
      skull.rotation.y += (targetRotationY - skull.rotation.y) * 0.15
    }
  })
}
</script>

<template>
  <div class="hero" @mousemove="handleMouseMove">
    <!-- Add digital background elements -->
    <div class="digital-background">
      <div class="grid"></div>
      <div class="digital-particles"></div>
    </div>
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

    <!-- Desktop Layout -->
    <template v-if="!isMobile">
      <div class="skull-container">
        <!-- Three.js skull renders here -->
      </div>
      <div class="content-grid desktop">
        <div class="section-1 content-section" :style="{ '--rotate-angle': '-5deg' }">
          <h2>Create</h2>
          <p>Build something extraordinary</p>
        </div>
        <div class="section-2 content-section" :style="{ '--rotate-angle': '5deg' }">
          <h2>Innovate</h2>
          <p>Push the boundaries</p>
        </div>
        <div class="section-3 content-section" :style="{ '--rotate-angle': '-3deg' }">
          <h2>Transform</h2>
          <p>Change the game</p>
        </div>
        <div class="section-4 content-section" :style="{ '--rotate-angle': '3deg' }">
          <h2>Evolve</h2>
          <p>Stay ahead of tomorrow</p>
        </div>
      </div>
    </template>

    <!-- Mobile Layout -->
    <template v-else>
      <div class="content-grid mobile">
        <div
          v-for="(section, index) in sections"
          :key="section.title"
          class="content-section mobile"
          :style="{ '--index': index }"
        >
          <h2>{{ section.title }}</h2>
          <p>{{ section.description }}</p>
        </div>
      </div>
    </template>
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
  --primary-color: #0088ff;
  --dark-color: #005299;
  --light-color: #40a9ff;
}

html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
  background: #000;
  scroll-behavior: smooth;
}

.hero {
  position: fixed;
  inset: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background: radial-gradient(circle at center, rgba(0, 0, 0, 0.8) 0%, #000 100%);
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
  background-color: var(--primary-color);
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
  background-color: var(--dark-color);
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
  background: linear-gradient(90deg, var(--primary-color), var(--light-color));
  animation: spread 0.8s ease-out forwards;
  box-shadow:
    0 0 10px var(--light-color),
    0 0 20px var(--primary-color);
  opacity: 0;
}

.circuit-branch {
  position: absolute;
  left: var(--distance);
  width: 30%;
  height: 2px;
  background: linear-gradient(90deg, var(--primary-color), transparent);
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
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 600px;
  height: 600px;
  display: flex;
  justify-content: center;
  align-items: center;
  opacity: 0;
  transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 80;
  perspective: 1200px;
}

.skull-container::after {
  display: none;
}

.skull-container.show {
  opacity: 1;
}

.skull-container.show::after {
  opacity: 1;
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
  padding: 3rem;
  width: 400px;
  background: rgba(0, 136, 255, 0.1);
  border: 1px solid rgba(64, 169, 255, 0.3);
  border-radius: 12px;
  backdrop-filter: blur(8px);
  opacity: 0;
  transform: translateY(20px) rotate(var(--rotate-angle));
  transition: all 0.3s ease;
  pointer-events: auto;
  cursor: pointer;
  z-index: 90;
  box-shadow:
    0 0 30px rgba(0, 136, 255, 0.1),
    inset 0 0 20px rgba(0, 136, 255, 0.05);
  position: relative;
  overflow: hidden;
}

.content-section.show {
  opacity: 1;
  transform: translateY(0) rotate(var(--rotate-angle));
}

.content-section h2 {
  color: var(--light-color);
  font-size: 2.5rem;
  margin-bottom: 1rem;
}

.content-section p {
  color: white;
  opacity: 0.8;
  font-size: 1.2rem;
  line-height: 1.6;
}

/* Update section positions */
.section-1 {
  top: 15vh;
  left: 15%;
  --rotate-angle: -5deg;
}

.section-2 {
  top: 15vh;
  right: 15%;
  --rotate-angle: 5deg;
}

.section-3 {
  bottom: 15vh;
  left: 15%;
  --rotate-angle: -3deg;
}

.section-4 {
  bottom: 15vh;
  right: 15%;
  --rotate-angle: 3deg;
}

/* Update hover effect */
.content-section:hover {
  transform: scale(1.03) rotate(0deg) !important;
  background: rgba(0, 136, 255, 0.15);
  border-color: var(--light-color);
  box-shadow:
    0 0 30px rgba(0, 136, 255, 0.2),
    inset 0 0 20px rgba(0, 136, 255, 0.1);
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
  width: 400px;
  height: 200px;
  border: 2px solid var(--light-color);
  opacity: 0.5;
  transform-origin: center;
  filter: drop-shadow(0 0 20px rgba(159, 122, 234, 0.2));
}

.progress-arc.top {
  border-radius: 200px 200px 0 0;
  border-bottom: none;
  top: calc(50% - 280px);
  animation: arcProgress 4s ease-in-out forwards;
}

.progress-arc.bottom {
  border-radius: 0 0 200px 200px;
  border-top: none;
  bottom: calc(50% - 280px);
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
  border: 1px solid rgba(64, 169, 255, 0.2);
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
  background: linear-gradient(90deg, transparent, rgba(64, 169, 255, 0.2), transparent);
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
  border-color: rgba(64, 169, 255, 0.4);
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
  background: linear-gradient(90deg, #fff, var(--light-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 0 10px rgba(64, 169, 255, 0.3));
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.site-header:hover .brand {
  filter: drop-shadow(0 0 15px rgba(64, 169, 255, 0.5));
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
  background: linear-gradient(120deg, #fff, var(--light-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 0 30px rgba(64, 169, 255, 0.3));
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
    filter: drop-shadow(0 0 30px rgba(64, 169, 255, 0.3));
    transform: scale(1);
  }
  50% {
    filter: drop-shadow(0 0 50px rgba(64, 169, 255, 0.5));
    transform: scale(1.02);
  }
}

/* Add menu icon styles */
.menu-icon {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding-left: 1rem;
  margin-left: 1rem;
  border-left: 1px solid rgba(64, 169, 255, 0.2);
}

.menu-icon span {
  display: block;
  width: 20px;
  height: 2px;
  background: var(--light-color);
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
  border: 1px solid rgba(64, 169, 255, 0.2);
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
  background: linear-gradient(90deg, var(--primary-color), transparent);
  opacity: 0;
  transition: all 0.3s ease;
}

.menu-item:hover .menu-highlight {
  opacity: 1;
  transform: translateX(0);
}

.menu-item:hover {
  color: var(--light-color);
}

/* Update header hover states */
.header-content.menu-active {
  border-color: var(--light-color);
  background: rgba(0, 0, 0, 0.6);
  border-radius: 24px;
}

/* Add responsive styles */
@media (max-width: 768px) {
  .content-grid.mobile {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 1rem;
    min-height: 100vh;
    padding-top: 5rem;
    background: linear-gradient(135deg, rgba(64, 169, 255, 0.1) 0%, rgba(0, 0, 0, 0.95) 100%);
    overflow-y: auto;
  }

  .content-section.mobile {
    position: relative;
    width: 90%;
    max-width: 350px;
    margin: 1rem 0;
    padding: 2rem;
    opacity: 0;
    transform: translateX(-50px);
    background: rgba(0, 136, 255, 0.1);
    border: 1px solid rgba(64, 169, 255, 0.3);
    border-radius: 16px;
    backdrop-filter: blur(10px);
    box-shadow:
      0 4px 6px rgba(0, 0, 0, 0.1),
      0 1px 3px rgba(0, 0, 0, 0.08);
  }

  .content-section.mobile.show {
    animation: slideInMobile 0.6s cubic-bezier(0.4, 0, 0.2, 1) forwards;
    animation-delay: calc(var(--index) * 0.15s);
  }

  @keyframes slideInMobile {
    0% {
      opacity: 0;
      transform: translateX(-50px);
    }
    100% {
      opacity: 1;
      transform: translateX(0);
    }
  }

  .content-section.mobile:nth-child(even) {
    transform: translateX(50px);
  }

  .content-section.mobile:nth-child(even).show {
    animation-name: slideInMobileRight;
  }

  @keyframes slideInMobileRight {
    0% {
      opacity: 0;
      transform: translateX(50px);
    }
    100% {
      opacity: 1;
      transform: translateX(0);
    }
  }

  .content-section.mobile:active {
    transform: scale(0.98);
    background: rgba(0, 136, 255, 0.2);
    border-color: var(--light-color);
  }

  .content-section.mobile h2 {
    font-size: 1.75rem;
    margin-bottom: 0.75rem;
    background: linear-gradient(120deg, #fff, var(--light-color));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .content-section.mobile p {
    font-size: 1rem;
    line-height: 1.5;
    color: rgba(255, 255, 255, 0.9);
  }

  /* Add floating effect */
  .content-section.mobile.show {
    animation:
      slideInMobile 0.6s cubic-bezier(0.4, 0, 0.2, 1) forwards,
      float 3s ease-in-out infinite;
    animation-delay: calc(var(--index) * 0.15s), 0.6s;
  }

  @keyframes float {
    0%,
    100% {
      transform: translateY(0);
    }
    50% {
      transform: translateY(-10px);
    }
  }

  /* Adjust skull container for mobile */
  .skull-container {
    width: 100vw;
    height: 100vw;
    max-width: 400px;
    max-height: 400px;
  }

  /* Adjust content grid layout */
  .content-grid {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 1rem;
    overflow-y: auto;
    pointer-events: all;
    padding-top: 6rem;
  }

  .content-section {
    position: relative;
    top: unset;
    left: unset;
    right: unset;
    bottom: unset;
    width: 90%;
    max-width: 350px;
    margin: 1rem 0;
    transform: translateY(20px);
    --rotate-angle: 0deg;
  }

  /* Adjust intro text */
  .intro-text .brand {
    font-size: 3rem;
  }

  .intro-text .welcome {
    font-size: 1rem;
  }

  /* Adjust progress arcs */
  .progress-arc {
    width: 280px;
    height: 140px;
  }

  .progress-arc.top {
    top: calc(50% - 200px);
  }

  .progress-arc.bottom {
    bottom: calc(50% - 200px);
  }

  /* Adjust header for mobile */
  .site-header {
    width: 90%;
    top: 1rem;
  }

  .header-content {
    width: 100%;
    padding: 0.5rem 1rem;
  }

  .site-header .brand {
    font-size: 1.25rem;
  }

  /* Adjust menu dropdown */
  .menu-dropdown {
    width: 100%;
    left: 0;
    transform: translateY(-10px);
    border-radius: 12px;
  }

  .menu-dropdown.show {
    transform: translateY(0);
  }
}

/* Add tablet-specific adjustments */
@media (min-width: 769px) and (max-width: 1024px) {
  .skull-container {
    width: 600px;
    height: 600px;
  }

  .content-section {
    width: 250px;
  }

  .intro-text .brand {
    font-size: 4rem;
  }
}

/* Add touch device optimizations */
@media (hover: none) {
  .content-section {
    transition: transform 0.3s ease;
  }

  .content-section:active {
    transform: scale(1.02);
  }
}

/* Desktop-specific styles */
.content-grid.desktop {
  position: fixed;
  inset: 0;
  pointer-events: none;
  padding: 2rem;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

.content-grid.desktop .content-section {
  position: absolute;
  width: 350px;
  padding: 2.5rem;
  margin: 1rem;
  background: rgba(0, 136, 255, 0.1);
  border: 1px solid rgba(64, 169, 255, 0.3);
  border-radius: 12px;
  backdrop-filter: blur(8px);
  transform: rotate(var(--rotate-angle));
  transition: all 0.3s ease;
  pointer-events: auto;
  opacity: 0;
}

.content-grid.desktop .content-section.show {
  opacity: 1;
}

/* Keep your existing section positioning */
.section-1 {
  top: 15vh;
  left: 15%;
}
.section-2 {
  top: 15vh;
  right: 15%;
}
.section-3 {
  bottom: 15vh;
  left: 15%;
}
.section-4 {
  bottom: 15vh;
  right: 15%;
}

/* Add these new styles */
.digital-background {
  position: fixed;
  inset: 0;
  z-index: 1;
  overflow: hidden;
  pointer-events: none;
}

.grid {
  position: absolute;
  inset: -100%;
  background-image:
    linear-gradient(to right, rgba(0, 136, 255, 0.1) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(0, 136, 255, 0.1) 1px, transparent 1px);
  background-size: 50px 50px;
  transform: perspective(500px) rotateX(60deg);
  animation: gridMove 20s linear infinite;
}

.digital-particles {
  position: absolute;
  inset: 0;
}

.digital-particles::before {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background-image: radial-gradient(circle at center, var(--light-color) 1px, transparent 1px);
  background-size: 50px 50px;
  animation: particlesFade 3s linear infinite;
  opacity: 0.3;
  mask-image: linear-gradient(
    to top,
    rgba(0, 0, 0, 1) 0%,
    rgba(0, 0, 0, 1) 30%,
    rgba(0, 0, 0, 0.8) 45%,
    rgba(0, 0, 0, 0.4) 60%,
    rgba(0, 0, 0, 0) 80%
  );
  -webkit-mask-image: linear-gradient(
    to top,
    rgba(0, 0, 0, 1) 0%,
    rgba(0, 0, 0, 1) 30%,
    rgba(0, 0, 0, 0.8) 45%,
    rgba(0, 0, 0, 0.4) 60%,
    rgba(0, 0, 0, 0) 80%
  );
}

.digital-particles::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background-image:
    radial-gradient(circle at 30% 70%, var(--primary-color) 1px, transparent 1px),
    radial-gradient(circle at 70% 30%, var(--glow) 1px, transparent 1px);
  background-size:
    100px 100px,
    80px 80px;
  animation: particlesMove 8s linear infinite;
  opacity: 0.2;
  mask-image: linear-gradient(
    to top,
    rgba(0, 0, 0, 1) 0%,
    rgba(0, 0, 0, 1) 25%,
    rgba(0, 0, 0, 0.8) 40%,
    rgba(0, 0, 0, 0.4) 55%,
    rgba(0, 0, 0, 0) 75%
  );
  -webkit-mask-image: linear-gradient(
    to top,
    rgba(0, 0, 0, 1) 0%,
    rgba(0, 0, 0, 1) 25%,
    rgba(0, 0, 0, 0.8) 40%,
    rgba(0, 0, 0, 0.4) 55%,
    rgba(0, 0, 0, 0) 75%
  );
}

@keyframes gridMove {
  0% {
    transform: perspective(500px) rotateX(60deg) translateY(0);
  }
  100% {
    transform: perspective(500px) rotateX(60deg) translateY(50px);
  }
}

@keyframes particlesFade {
  0%,
  100% {
    opacity: 0.3;
  }
  50% {
    opacity: 0.6;
  }
}

@keyframes particlesMove {
  0% {
    transform: translateY(0);
  }
  100% {
    transform: translateY(-100px);
  }
}

/* Update hero background */
.hero {
  background: radial-gradient(circle at center, rgba(0, 0, 0, 0.8) 0%, #000 100%);
}

/* Add subtle glow to content sections */
.content-section {
  box-shadow:
    0 0 30px rgba(0, 136, 255, 0.1),
    inset 0 0 20px rgba(0, 136, 255, 0.05);
}

/* Add these new hover effects for content sections */
.content-section {
  position: relative;
  overflow: hidden;
}

/* Add glitch effect container */
.content-section::before {
  content: '';
  position: absolute;
  inset: -2px;
  background: linear-gradient(90deg, transparent 33%, rgba(0, 136, 255, 0.3) 50%, transparent 67%);
  background-size: 300% 100%;
  opacity: 0;
  transition: all 0.3s ease;
  z-index: -1;
  animation: none;
}

/* Add distortion lines */
.content-section::after {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    transparent 0%,
    rgba(0, 136, 255, 0.1) 1px,
    transparent 2px,
    transparent 4px
  );
  opacity: 0;
  transition: all 0.3s ease;
  pointer-events: none;
}

/* Update hover effects */
.content-section:hover {
  transform: scale(1.03) rotate(0deg) !important;
  background: rgba(0, 136, 255, 0.15);
  border-color: var(--light-color);
  box-shadow:
    0 0 30px rgba(0, 136, 255, 0.2),
    inset 0 0 20px rgba(0, 136, 255, 0.1);
}

.content-section:hover::before {
  opacity: 1;
  animation: glitch 2s linear infinite;
}

.content-section:hover::after {
  opacity: 1;
  animation: scanline 1.5s linear infinite;
}

/* Add text glitch effect on hover */
.content-section:hover h2 {
  animation: textGlitch 0.3s ease infinite;
  text-shadow:
    2px 2px var(--primary-color),
    -2px -2px var(--glow);
}

/* Add glitch animations */
@keyframes glitch {
  0% {
    background-position: 100% 50%;
    transform: translateX(0);
  }
  25% {
    transform: translateX(-1px);
  }
  50% {
    background-position: 0% 50%;
    transform: translateX(1px);
  }
  75% {
    transform: translateX(-1px);
  }
  100% {
    background-position: 100% 50%;
    transform: translateX(0);
  }
}

@keyframes scanline {
  0% {
    transform: translateY(-100%);
  }
  100% {
    transform: translateY(100%);
  }
}

@keyframes textGlitch {
  0%,
  100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-1px) skewX(2deg);
  }
  75% {
    transform: translateX(1px) skewX(-2deg);
  }
}

/* Add distortion wave effect */
.content-section:hover {
  animation: distortionWave 2s ease infinite;
}

@keyframes distortionWave {
  0%,
  100% {
    clip-path: polygon(0% 0%, 100% 0%, 100% 100%, 0% 100%);
  }
  25% {
    clip-path: polygon(0% 0%, 100% 0%, 98% 100%, 2% 100%);
  }
  50% {
    clip-path: polygon(2% 0%, 98% 0%, 100% 100%, 0% 100%);
  }
  75% {
    clip-path: polygon(0% 0%, 100% 0%, 98% 100%, 2% 100%);
  }
}

/* Update content section base styles */
.content-section {
  position: absolute;
  width: 350px;
  padding: 2.5rem;
  margin: 1rem;
  transform-origin: center center;
}

/* Ensure hover doesn't cause overflow */
.content-section:hover {
  z-index: 100;
}
</style>
