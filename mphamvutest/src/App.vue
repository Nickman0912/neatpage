<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <div class="app">
    <nav class="navbar">
      <div class="nav-content">
        <router-link to="/" class="logo">Superlist</router-link>
        <div class="nav-links">
          <router-link to="/updates">Updates</router-link>
          <router-link to="/pricing">Pricing</router-link>
          <a href="#" class="sign-in">Sign in</a>
          <a href="#" class="download-btn">Download</a>
        </div>
      </div>
    </nav>
    <router-view></router-view>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
}

.navbar {
  padding: 1.5rem 2rem;
  position: fixed;
  width: 100%;
  top: 0;
  z-index: 100;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
}

.nav-content {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  font-size: 1.5rem;
  font-weight: bold;
  color: #000;
  text-decoration: none;
}

.nav-links {
  display: flex;
  gap: 2rem;
  align-items: center;
}

.nav-links a {
  text-decoration: none;
  color: #000;
  font-weight: 500;
}

.download-btn {
  background: #000;
  color: #fff !important;
  padding: 0.5rem 1rem;
  border-radius: 2rem;
}
</style>
