<template>
  <div class="main-container">
    <NuxtLink :to="backUrl" class="home-button">
      {{ backButtonText }}
    </NuxtLink>

    <div class="content-wrapper-single-post">
      <template v-if="pending">
        <UCard class="loading-card">
          <template #header>
            <h2 class="text-xl font-semibold">Завантаження посту...</h2>
          </template>
          <div class="text-center">
            <UProgress animation="carousel" class="mt-4" />
            <p class="mt-4 text-gray-600">Будь ласка, зачекайте.</p>
          </div>
        </UCard>
      </template>

      <template v-else-if="error || !post">
        <UCard class="not-found-card">
          <template #header>
            <h2 class="text-xl font-semibold error-title">Пост не знайдено</h2>
          </template>
          <p class="error-message">Схоже, запитуваної статті не існує або вона була видалена.</p>
          <template #footer>
            <UButton :to="backUrl" color="gray" size="md">{{ backButtonText }}</UButton>
          </template>
        </UCard>
      </template>

      <template v-else>
        <UCard class="post-card">
          <template #header>
            <div class="post-header">
              <h1 class="post-title">{{ post.title }}</h1>
              <div class="post-meta">
                <span v-if="post.user" class="post-author">Автор: <strong>{{ post.user.name }}</strong></span>
                <span v-if="post.category" class="post-category">Категорія: <strong>{{ post.category.title }}</strong></span>
                <span v-if="post.is_published && post.published_at" class="post-published-at">Опубліковано: <strong>{{ formatDate(post.published_at) }}</strong></span>
                <UBadge :color="post.is_published ? 'emerald' : 'orange'" variant="subtle" size="lg">
                  {{ post.is_published ? 'Опубліковано' : 'Чернетка' }}
                </UBadge>
              </div>
            </div>
          </template>

          <div class="post-content" v-html="post.content_html"></div>

          <template #footer>
            <div class="post-footer">
              <UButton icon="i-heroicons-arrow-left" :to="backUrl" color="gray" variant="ghost" size="lg">
                {{ backButtonText }}
              </UButton>
            </div>
          </template>
        </UCard>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useRoute } from 'vue-router';
import { useRuntimeConfig } from '#app';

interface Post {
  id: number;
  title: string;
  slug: string;
  content_html: string;
  content_raw: string;
  is_published: boolean;
  published_at: string | null;
  user: { name: string };
  category: { title: string };
}

const route = useRoute();
const config = useRuntimeConfig();
const postId = route.params.slug;

const sourceParam = route.query.source as string;

const backUrl = computed(() => {
  switch (sourceParam) {
    case 'custom':
      return '/BlogPosts';
    case 'nuxt-ui':
      return '/BlogPostsUI';
    default:
      return '/blog';
  }
});

const backButtonText = computed(() => {
  switch (sourceParam) {
    case 'custom':
      return 'Повернутися до звичайної таблиці';
    case 'nuxt-ui':
      return 'Повернутися до NuxtUI таблиці';
    default:
      return 'Повернутися до списку';
  }
});

const post = ref<Post | null>(null);

const { data, pending, error } = await useAsyncData<Post | null>(
    `blog-post-${postId}`,
    async () => {
      try {
        const apiUrl = `${config.public.apiBase}/blog/posts/${postId}`;
        console.log("Fetching single post from URL:", apiUrl);
        const response = await $fetch<Post>(apiUrl);
        return response;
      } catch (e: any) {
        console.error("Error fetching post:", e);
        if (e.statusCode === 404) {
          return null;
        }
        throw e;
      }
    }
);

if (data.value) {
  post.value = data.value;
}

const formatDate = (dateString: string | null): string => {
  if (!dateString) return '';
  try {
    const date = new Date(dateString);
    return new Intl.DateTimeFormat('uk-UA', {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit'
    }).format(date);
  } catch (e) {
    return '';
  }
};
</script>

<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f0f2f5;
  padding: 30px;
  box-sizing: border-box;
  position: relative;
}

.home-button {
  position: absolute;
  top: 30px;
  left: 30px;
  display: inline-block;
  padding: 10px 20px;
  background-color: #6c757d;
  color: white;
  text-decoration: none;
  border-radius: 8px;
  font-weight: 600;
  transition: background-color 0.3s ease, transform 0.2s ease;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
  z-index: 10;
}

.home-button:hover {
  background-color: #5a6268;
  transform: translateY(-2px);
}

.home-button:active {
  background-color: #495057;
  transform: translateY(0);
}

.content-wrapper-single-post {
  max-width: 900px;
  width: 100%;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 40px;
  box-sizing: border-box;
}

.post-card {
  box-shadow: none;
  border: none;
  background-color: transparent;
}

.post-card :deep(.prose h1),
.post-card :deep(.prose h2),
.post-card :deep(.prose h3) {
  font-family: 'Montserrat', sans-serif;
  color: #2c3e50;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.08);
}

.post-header {
  text-align: center;
  padding-bottom: 25px;
  border-bottom: 1px solid #e0e0e0;
  margin-bottom: 25px;
}

.post-title {
  font-family: 'Montserrat', sans-serif;
  font-size: 3em;
  font-weight: 700;
  margin-bottom: 15px;
  color: #2c3e50;
  line-height: 1.2;
}

.post-meta {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 1.5rem;
  font-size: 0.95em;
  color: #6a6a6a;
  font-family: 'Open Sans', sans-serif;
}

.post-meta span {
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.post-meta strong {
  font-weight: 600;
  color: #444;
}

.post-content {
  padding: 0 20px;
  line-height: 1.7;
  font-size: 1.1em;
  color: #374151;
  font-family: 'Open Sans', sans-serif;
}

.post-content :deep(p) {
  margin-bottom: 1.2em;
}

.post-content :deep(img) {
  max-width: 100%;
  height: auto;
  border-radius: 8px;
  margin: 1.5rem auto;
  display: block;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
}

.post-content :deep(h1),
.post-content :deep(h2),
.post-content :deep(h3),
.post-content :deep(h4),
.post-content :deep(h5),
.post-content :deep(h6) {
  font-family: 'Montserrat', sans-serif;
  color: #2c3e50;
  margin-top: 1.8em;
  margin-bottom: 0.8em;
  font-weight: 600;
  line-height: 1.3;
}

.post-content :deep(h2) { font-size: 1.8em; }
.post-content :deep(h3) { font-size: 1.5em; }
.post-content :deep(h4) { font-size: 1.3em; }

.post-content :deep(ul),
.post-content :deep(ol) {
  margin-left: 25px;
  margin-bottom: 1.2em;
}

.post-content :deep(li) {
  margin-bottom: 0.5em;
}

.post-content :deep(a) {
  color: #3498db;
  text-decoration: none;
  transition: color 0.3s ease;
}

.post-content :deep(a:hover) {
  color: #2980b9;
  text-decoration: underline;
}

.post-content :deep(blockquote) {
  border-left: 4px solid #3498db;
  padding-left: 20px;
  margin: 1.5em 0;
  color: #555;
  font-style: italic;
  background-color: #f8f9fa;
  padding: 15px 20px;
  border-radius: 5px;
}

.post-footer {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding-top: 25px;
  border-top: 1px solid #e0e0e0;
  margin-top: 25px;
}

.loading-card, .not-found-card {
  max-width: 500px;
  width: 100%;
  margin: 2rem auto;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 30px;
  text-align: center;
  border: none;
}

.loading-card :deep(.ubutton) {
  background-color: #3498db;
  color: white;
  border-radius: 8px;
  font-weight: 600;
  transition: background-color 0.3s ease, transform 0.2s ease;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.loading-card :deep(.ubutton:hover) {
  background-color: #2980b9;
  transform: translateY(-3px);
}

.loading-card :deep(.ubutton:active) {
  background-color: #1f618d;
  transform: translateY(0);
}

.loading-card .text-xl, .not-found-card .text-xl {
  font-family: 'Montserrat', sans-serif;
  color: #2c3e50;
  font-weight: 700;
  margin-bottom: 1rem;
}

.loading-card p, .not-found-card p {
  font-family: 'Open Sans', sans-serif;
  font-size: 1.1em;
  color: #555;
}

.error-title {
  color: #dc3545;
}

.error-message {
  color: #6a6a6a;
  margin-bottom: 1.5rem;
}
</style>