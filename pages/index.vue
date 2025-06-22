<template>
  <div class="main-container">
    <h1>Anpilohov Blog</h1>
    <div class="links-container">
      <NuxtLink to="/BlogPosts" class="link-card">
        <h2>Список постів з кастомною таблицею</h2>
        <p>Пости, оформлені у власному стилі.</p>
      </NuxtLink>
      <NuxtLink to="/BlogPostsUI" class="link-card">
        <h2>Список постів з NuxtUI таблицею</h2>
        <p>Пости, що використовують компоненти NuxtUI.</p>
      </NuxtLink>
      <NuxtLink to="/admin/blog/categories" class="link-card">
        <h2>Список категорій</h2>
        <p>Категорії, що використовують компоненти NuxtUI.</p>
      </NuxtLink>
    </div>
  </div>
</template>

<script setup>
</script>

<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f0f2f5;
  padding: 20px;
  box-sizing: border-box;
}

h1 {
  font-family: 'Montserrat', sans-serif;
  color: #2c3e50;
  text-align: center;
  margin-bottom: 40px;
  font-size: 4.5em;
  font-weight: 700;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.15);
}

.links-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 30px;
  max-width: 900px;
  width: 100%;
}

.link-card {
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
  padding: 30px;
  text-align: center;
  text-decoration: none;
  color: #333;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  flex: 1;
  min-width: 280px;
  max-width: 400px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.link-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.2);
}

.link-card h2 {
  font-family: 'Poppins', sans-serif;
  font-size: 1.8em;
  color: #3498db;
  margin-bottom: 15px;
  font-weight: 600;
}

.link-card p {
  font-family: 'Open Sans', sans-serif;
  font-size: 1em;
  color: #555;
  line-height: 1.6;
  margin-top: 0;
}
</style>