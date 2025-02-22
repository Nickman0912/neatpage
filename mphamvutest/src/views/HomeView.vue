<script setup lang="ts">
// Component logic goes here
</script>

<template>
  <div class="home">
    <section class="hero">
      <h1>Lists with Superpowers</h1>
      <p class="hero-subtitle">Brainstorm, note, plan and get more done with AI.</p>
      <div class="cta-buttons">
        <a href="#" class="primary-btn">Start today for free</a>
      </div>
      <div class="hero-video">
        <div class="placeholder-video">Video</div>
      </div>
    </section>

    <section class="features">
      <div class="feature-item">
        <h2>Tasks and notes together.</h2>
        <p>Turn your thoughts into actions.</p>
        <div class="feature-video">
          <div class="placeholder-video">Video</div>
        </div>
      </div>
    </section>

    <section class="feature-grid">
      <div class="feature-card">
        <h3>Natural Language</h3>
        <p>Type dates and set due dates into tasks.</p>
      </div>
      <div class="feature-card">
        <h3>Nested Tasks</h3>
        <p>Infinite level of subtasks</p>
      </div>
      <div class="feature-card">
        <h3>Tasks from Integrations</h3>
        <p>Slack messages, Linear tickets, & Github issues together</p>
      </div>
    </section>
  </div>
</template>

<style scoped>
.home {
  padding-top: 5rem;
}

.hero {
  text-align: center;
  padding: 4rem 2rem;
}

h1 {
  font-size: 4rem;
  font-weight: 700;
  margin-bottom: 1.5rem;
  background: linear-gradient(to right, #000, #333);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.hero-subtitle {
  font-size: 1.5rem;
  color: #666;
  margin-bottom: 2rem;
}

.primary-btn {
  background: #000;
  color: #fff;
  padding: 1rem 2rem;
  border-radius: 2rem;
  text-decoration: none;
  font-weight: 500;
  display: inline-block;
}

.hero-video {
  max-width: 1000px;
  margin: 4rem auto 0;
  border-radius: 1rem;
  overflow: hidden;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
}

.hero-video video {
  width: 100%;
  display: block;
}

.features {
  padding: 6rem 2rem;
}

.feature-item {
  max-width: 800px;
  margin: 0 auto;
  text-align: center;
}

.feature-item h2 {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.feature-item p {
  font-size: 1.5rem;
  color: #666;
  margin-bottom: 3rem;
}

.feature-video {
  border-radius: 1rem;
  overflow: hidden;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
}

.feature-video video {
  width: 100%;
  display: block;
}

.feature-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.feature-card {
  padding: 2rem;
  border-radius: 1rem;
  background: #f5f5f5;
}

.feature-card h3 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
}

.feature-card p {
  color: #666;
}

.placeholder-video {
  width: 100%;
  height: 400px;
  background-color: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  color: #666;
  border-radius: 8px;
}
</style>
