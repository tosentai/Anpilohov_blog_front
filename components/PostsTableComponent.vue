<template>
  <div class="main-container">
    <NuxtLink to="/" class="home-button">
      На головну
    </NuxtLink>
    <div class="content-wrapper">
      <h1>Список постів блогу</h1>

      <div v-if="loading" class="loading-indicator">
        <p>Завантаження постів...</p>
      </div>

      <div v-else>
        <div class="table-actions">
          <div class="posts-count">
            Всього постів: {{ posts.length }}
          </div>
        </div>

        <div class="table-card">
          <table class="styled-table">
            <thead>
            <tr>
              <th>#</th>
              <th>Автор</th>
              <th>Категорія</th>
              <th>Заголовок</th>
              <th>Дата публікації</th>
              <th>Статус</th>
              <th>Дії</th>
            </tr>
            </thead>
            <tbody>
            <tr
                v-for="post in posts"
                :key="post.id"
                :class="{ 'unpublished-row': !post.is_published, 'clickable-row': true }"
                @click="navigateToPost(post)"
            >
              <td data-label="#">{{ post.id }}</td>
              <td data-label="Автор">{{ post.user.name }}</td>
              <td data-label="Категорія">{{ post.category.title }}</td>
              <td data-label="Заголовок" class="post-title-cell">
                <span class="post-title">{{ post.title }}</span>
              </td>
              <td data-label="Дата публікації">
                {{ post.published_at ? new Date(post.published_at).toLocaleDateString('uk-UA', { year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' }) : 'Не опубліковано' }}
              </td>
              <td data-label="Статус">
                <span :class="{'status-published': post.is_published, 'status-unpublished': !post.is_published}">
                  {{ post.is_published ? 'Опубліковано' : 'Чернетка' }}
                </span>
              </td>
              <td data-label="Дії" class="actions-cell" @click.stop>
                <button
                    @click="editPost(post.id)"
                    class="edit-button"
                    title="Редагувати пост"
                >
                  Редагувати
                </button>
              </td>
            </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

interface Post {
  id: number;
  title: string;
  slug: string;
  is_published: boolean;
  published_at: string | null;
  user_id: number;
  category_id: number;
  user: { name: string };
  category: { title: string };
}

const posts = ref<Post[]>([]);
const loading = ref(false);
const config = useRuntimeConfig();

const getPosts = async () => {
  loading.value = true;
  try {
    let allPosts: Post[] = [];
    let currentPage = 1;
    let totalPages = 1;

    do {
      const apiUrl = `${config.public.apiBase}/blog/posts?page=${currentPage}&per_page=50`;
      console.log(`Завантажуємо сторінку ${currentPage} з ${apiUrl}`);

      const response = await $fetch<{
        data: Post[],
        meta: {
          current_page: number,
          last_page: number,
          total: number
        }
      }>(apiUrl);

      allPosts = [...allPosts, ...response.data];
      totalPages = response.meta.last_page;
      currentPage++;

      console.log(`Завантажено ${response.data.length} постів з сторінки ${response.meta.current_page}`);
      console.log(`Всього сторінок: ${totalPages}, поточна сторінка: ${response.meta.current_page}`);

    } while (currentPage <= totalPages);

    console.log(`Всього завантажено постів: ${allPosts.length}`);
    posts.value = allPosts;
  } catch (error) {
    console.error("Error fetching posts:", error);
  } finally {
    loading.value = false;
  }
};

const navigateToPost = async (post: Post) => {
  try {
    console.log('Navigating to:', `/blog/${post.slug}?source=custom`);
    await navigateTo(`/blog/${post.slug}?source=custom`);
  } catch (error) {
    console.error('Navigation error:', error);
    window.location.href = `/blog/${post.slug}?source=custom`;
  }
};

const editPost = async (postId: number) => {
  try {
    console.log('Navigating to edit:', `/admin/blog/posts/${postId}/edit`);
    await navigateTo(`/admin/blog/posts/${postId}/edit`);
  } catch (error) {
    console.error('Edit navigation error:', error);
    window.location.href = `/admin/blog/posts/${postId}/edit`;
  }
};

onMounted(() => {
  getPosts();
});
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

.content-wrapper {
  max-width: 1200px;
  width: 100%;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 40px;
}

h1 {
  font-family: 'Montserrat', sans-serif;
  color: #2c3e50;
  text-align: center;
  margin-bottom: 40px;
  font-size: 3em;
  font-weight: 700;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
}

.loading-indicator {
  text-align: center;
  margin: 40px 0;
}

.loading-indicator p {
  font-size: 1.2em;
  color: #6c757d;
}

.table-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

.posts-count {
  font-weight: 600;
  color: #6c757d;
  font-size: 0.9em;
}

.styled-button {
  display: inline-block;
  padding: 12px 25px;
  background-color: #3498db;
  color: white;
  text-decoration: none;
  border-radius: 8px;
  font-weight: 600;
  transition: background-color 0.3s ease, transform 0.2s ease;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.styled-button:hover {
  background-color: #2980b9;
  transform: translateY(-3px);
}

.styled-button:active {
  background-color: #1f618d;
  transform: translateY(0);
}

.table-card {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.05);
  overflow: hidden;
  border: 1px solid #e0e0e0;
}

.styled-table {
  width: 100%;
  border-collapse: collapse;
  font-family: 'Open Sans', sans-serif;
}

.styled-table thead {
  background-color: #f8f9fa;
  border-bottom: 2px solid #e0e0e0;
}

.styled-table th {
  text-align: left;
  padding: 15px 25px;
  font-size: 0.95em;
  font-weight: 700;
  color: #555;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.styled-table td {
  padding: 15px 25px;
  font-size: 0.9em;
  color: #444;
  vertical-align: middle;
  border-bottom: 1px solid #f0f0f0;
}

.styled-table tbody tr:last-child td {
  border-bottom: none;
}

.clickable-row {
  cursor: pointer;
  transition: background-color 0.2s ease, transform 0.1s ease;
}

.clickable-row:hover {
  background-color: #f0f8ff;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(52, 152, 219, 0.1);
}

.unpublished-row {
  background-color: #fff9e6;
  color: #6a6a6a;
}

.unpublished-row:hover {
  background-color: #fff3d9;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(255, 193, 7, 0.1);
}

.post-title-cell {
  position: relative;
}

.post-title {
  color: #444;
  font-weight: 500;
  cursor: pointer;
  transition: color 0.3s ease;
}

.post-title:hover {
  color: #333;
}

.unpublished-row .post-title {
  color: #6a6a6a;
}

.unpublished-row .post-title:hover {
  color: #5a5a5a;
}

.status-published {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 0.8em;
  font-weight: 600;
  white-space: nowrap;
  display: inline-block;
  text-align: center;
  min-width: 80px;
}

.status-unpublished {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 0.8em;
  font-weight: 600;
  white-space: nowrap;
  display: inline-block;
  text-align: center;
  min-width: 80px;
}

.actions-cell {
  text-align: center;
}

.edit-button {
  padding: 6px 12px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 0.8em;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.edit-button:hover {
  background-color: #218838;
  transform: translateY(-1px);
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.15);
}

.edit-button:active {
  background-color: #1e7e34;
  transform: translateY(0);
}
</style>