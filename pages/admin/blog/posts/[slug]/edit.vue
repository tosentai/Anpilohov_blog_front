<template>
  <div class="flex flex-col items-center justify-center min-h-screen p-4 bg-gray-100 relative">
    <NuxtLink
        to="/BlogPostsUI"
        class="absolute top-4 left-4 inline-flex items-center px-4 py-2 bg-gray-600 text-white text-sm font-semibold rounded-lg shadow-md transition-all duration-300 ease-in-out hover:bg-gray-700 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-opacity-75"
    >
      Назад до списку
    </NuxtLink>

    <div class="w-full max-w-screen-md bg-white rounded-xl shadow-lg p-6">
      <div class="text-center mb-6">
        <h1 class="font-['Montserrat'] text-3xl font-extrabold text-gray-800 mb-2 drop-shadow-md">
          Редагувати блог-пост
        </h1>
        <p class="text-gray-600 text-base">{{ post.title }}</p>
      </div>

      <div v-if="loading" class="text-center py-8">
        <div class="inline-flex items-center px-4 py-2 bg-blue-50 text-blue-700 rounded-lg">
          <div class="animate-spin rounded-full h-4 w-4 border-b-2 border-blue-700 mr-2"></div>
          Завантаження даних...
        </div>
      </div>

      <div v-else-if="error" class="mb-6 p-4 bg-red-50 border border-red-200 rounded-lg">
        <p class="text-red-800 font-medium">{{ error }}</p>
        <div v-if="categoryErrorDetails" class="mt-2 text-sm text-red-600">
          Деталі помилки категорій: {{ categoryErrorDetails }}
        </div>
      </div>

      <!-- Form -->
      <form @submit.prevent="updatePost" v-else class="space-y-4">
        <div>
          <label for="title" class="block text-sm font-medium text-gray-700 mb-1">
            Заголовок поста <span class="text-red-500">*</span>
          </label>
          <input
              v-model="post.title"
              type="text"
              id="title"
              name="title"
              required
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors text-sm"
              placeholder="Введіть заголовок поста"
              :class="{ 'border-red-500': validationErrors.title }"
          />
          <p v-if="validationErrors.title" class="mt-1 text-xs text-red-600">{{ validationErrors.title[0] }}</p>
        </div>

        <div>
          <label for="category_id" class="block text-sm font-medium text-gray-700 mb-1">
            Категорія <span class="text-red-500">*</span>
          </label>
          <select
              v-model="post.category_id"
              id="category_id"
              name="category_id"
              required
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors text-sm"
              :class="{ 'border-red-500': validationErrors.category_id }"
          >
            <option :value="null" disabled>Оберіть категорію</option>
            <option v-for="category in categories" :key="category.id" :value="category.id">
              {{ category.title }}
            </option>
          </select>
          <p v-if="validationErrors.category_id" class="mt-1 text-xs text-red-600">{{ validationErrors.category_id[0] }}</p>
        </div>

        <div>
          <label for="excerpt" class="block text-sm font-medium text-gray-700 mb-1">
            Короткий опис
          </label>
          <textarea
              v-model="post.excerpt"
              id="excerpt"
              name="excerpt"
              rows="2"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors resize-y text-sm"
              placeholder="Введіть короткий опис поста (необов'язково)"
              :class="{ 'border-red-500': validationErrors.excerpt }"
          ></textarea>
          <p v-if="validationErrors.excerpt" class="mt-1 text-xs text-red-600">{{ validationErrors.excerpt[0] }}</p>
          <p class="mt-1 text-xs text-gray-500">Максимум 500 символів</p>
        </div>

        <div>
          <label for="content_raw" class="block text-sm font-medium text-gray-700 mb-1">
            Контент поста <span class="text-red-500">*</span>
          </label>
          <textarea
              v-model="post.content_raw"
              id="content_raw"
              name="content_raw"
              rows="8"
              required
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors resize-y font-mono text-xs"
              placeholder="Введіть контент поста (підтримується Markdown)"
              :class="{ 'border-red-500': validationErrors.content_raw }"
          ></textarea>
          <p v-if="validationErrors.content_raw" class="mt-1 text-xs text-red-600">{{ validationErrors.content_raw[0] }}</p>
          <p class="mt-1 text-xs text-gray-500">Ви можете використовувати Markdown для форматування</p>
        </div>

        <div>
          <label for="published_at" class="block text-sm font-medium text-gray-700 mb-1">
            Дата публікації
          </label>
          <input
              v-model="post.published_at"
              type="datetime-local"
              id="published_at"
              name="published_at"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors text-sm"
              :class="{ 'border-red-500': validationErrors.published_at }"
          />
          <p v-if="validationErrors.published_at" class="mt-1 text-xs text-red-600">{{ validationErrors.published_at[0] }}</p>
          <p class="mt-1 text-xs text-gray-500">Залиште порожнім для публікації зараз</p>
        </div>

        <div class="flex items-center">
          <input
              v-model="post.is_published"
              type="checkbox"
              id="is_published"
              name="is_published"
              class="h-3 w-3 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
          />
          <label for="is_published" class="ml-2 block text-sm text-gray-700">
            Опублікувати пост відразу
          </label>
          <p v-if="validationErrors.is_published" class="ml-2 text-xs text-red-600">{{ validationErrors.is_published[0] }}</p>
        </div>

        <div class="flex justify-end gap-3 pt-4 border-t border-gray-200">
          <button
              type="button"
              @click="goBack"
              class="px-4 py-2 bg-gray-200 text-gray-800 text-sm font-semibold rounded-lg hover:bg-gray-300 transition-colors"
          >
            Скасувати
          </button>
          <button
              type="submit"
              class="px-4 py-2 bg-blue-500 text-white text-sm font-semibold rounded-lg hover:bg-blue-600 transition-colors"
          >
            Оновити пост
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useRuntimeConfig } from '#app';

const route = useRoute();
const router = useRouter();
const config = useRuntimeConfig();

const post = ref({
  title: '',
  content_raw: '',
  excerpt: '',
  category_id: null,
  is_published: false,
  published_at: null,
});
const categories = ref([]);
const loading = ref(true);
const error = ref(null);
const categoryErrorDetails = ref(null);
const validationErrors = ref({});

const fetchPost = async (slug) => {
  try {
    const response = await axios.get(`${config.public.apiBase}/blog/posts/${slug}`);
    post.value = {
      ...response.data,
      published_at: response.data.published_at
          ? new Date(response.data.published_at).toISOString().slice(0, 16)
          : null,
    };
    error.value = null;
  } catch (err) {
    console.error('Помилка при завантаженні посту:', err);
    error.value = 'Пост не знайдено або виникла помилка завантаження.';
  }
};

const fetchCategories = async () => {
  try {
    const response = await axios.get(`${config.public.apiBase}/blog/categories`);
    categories.value = response.data.data || response.data;
    categoryErrorDetails.value = null;
  } catch (err) {
    console.error('Помилка при завантаженні категорій:', err);
    error.value = 'Помилка при завантаженні категорій.';

    if (err.response) {
      console.error('Дані помилки відповіді:', err.response.data);
      console.error('Статус помилки:', err.response.status);
      console.error('Заголовки помилки:', err.response.headers);
      categoryErrorDetails.value = `Статус: ${err.response.status}, Дані: ${JSON.stringify(err.response.data)}`;
    } else if (err.request) {
      console.error('Немає відповіді від сервера:', err.request);
      categoryErrorDetails.value = 'Сервер не відповів.';
    } else {
      console.error('Помилка налаштування запиту:', err.message);
      categoryErrorDetails.value = `Помилка: ${err.message}`;
    }
  }
};

const updatePost = async () => {
  validationErrors.value = {};
  try {
    const slug = route.params.slug;
    const response = await axios.put(`${config.public.apiBase}/blog/posts/${slug}`, post.value);

    alert(response.data.message || 'Пост успішно оновлено!');
    await router.push('/BlogPostsUI');

  } catch (err) {
    if (err.response && err.response.status === 422) {
      validationErrors.value = err.response.data.errors;
    } else {
      console.error('Помилка при оновленні посту:', err);
      alert('Виникла помилка при оновленні посту. Спробуйте ще раз.');
    }
  }
};

const goBack = () => {
  router.go(-1);
};

onMounted(async () => {
  const slug = route.params.slug;
  await Promise.all([
    fetchPost(slug),
    fetchCategories()
  ]);
  loading.value = false;
});
</script>

<style scoped>
button, .transition-colors {
  transition: all 0.2s ease-in-out;
}

input:focus, textarea:focus, select:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.font-mono {
  font-family: 'Courier New', Courier, monospace;
}
</style>