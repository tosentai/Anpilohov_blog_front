<template>
  <div class="main-container">
    <NuxtLink to="/" class="home-button">
      На головну
    </NuxtLink>

    <div class="content-wrapper">
      <div class="card-header">
        <h1>Список постів (NuxtUI Таблиця)</h1>
        <div class="table-actions">
          <NuxtLink to="/admin/blog/posts/create" class="styled-button">
            Додати новий пост
          </NuxtLink>
        </div>
      </div>

      <div class="table-card">
        <div class="table-wrapper">
          <template v-if="pending">
            <div class="loading-state">Завантаження постів...</div>
          </template>
          <template v-else-if="error">
            <div class="error-state">Помилка завантаження постів: {{ error.message }}</div>
          </template>
          <template v-else-if="posts.length === 0">
            <div class="empty-state">Постів поки що немає.</div>
          </template>
          <template v-else>
            <UTable
                ref="table"
                v-model:pagination="pagination"
                :data="posts"
                :columns="columns"
                :pagination-options="{ getPaginationRowModel: getPaginationRowModel() }"
                class="styled-table"
                :thead-class="'table-thead'"
                :tbody-tr-class="(row) => getRowClass(row)"
                :tbody-td-class="'table-cell'"
                :th-class="'table-th'"
            />
          </template>
        </div>

        <div class="footer-pagination" v-if="!pending && !error && posts.length > 0">
          <div class="pagination-control">
            <div class="pagination-label">
              Показано {{ ((pagination.pageIndex) * pagination.pageSize) + 1 }} -
              {{ Math.min((pagination.pageIndex + 1) * pagination.pageSize, posts.length) }}
              з {{ posts.length }} результатів
            </div>
            <UPagination
                :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
                :items-per-page="table?.tableApi?.getState().pagination.pageSize"
                :total="table?.tableApi?.getFilteredRowModel().rows.length"
                @update:page="p => table?.tableApi?.setPageIndex(p - 1)"
                class="custom-pagination"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, h } from 'vue';
import { getPaginationRowModel } from '@tanstack/vue-table';
import type { TableColumn } from '@nuxt/ui';
import { NuxtLink } from '#components';

const table = ref();

type Post = {
  id: string;
  name: string;
  category: string;
  title: string;
  date: string;
  is_published: boolean;
};

const posts = ref<Post[]>([]);
const config = useRuntimeConfig();

const getRowClass = (row: any) => {
  const isPublished = row.original.is_published;
  return isPublished ? 'table-row published-row' : 'table-row unpublished-row';
};

const { data, pending, error, refresh } = await useAsyncData(
    'all-blog-posts',
    async () => {
      let allPosts: Post[] = [];
      let currentPage = 1;
      let lastPage = 1;

      do {
        const response: any = await $fetch(`${config.public.apiBase}/blog/posts?page=${currentPage}`);
        const pagePosts = response.data.map((post: any) => ({
          id: String(post.id),
          name: post.user?.name || '',
          category: post.category?.title || '',
          title: post.title,
          date: post.published_at,
          is_published: post.is_published
        })) as Post[];

        allPosts = allPosts.concat(pagePosts);
        lastPage = response.meta.last_page;
        currentPage++;
      } while (currentPage <= lastPage);

      return allPosts;
    }
);

if (data.value) {
  posts.value = data.value;
}

const columns: TableColumn<Post>[] = [
  {
    accessorKey: 'id',
    header: '#',
    cell: ({ row }) => `#${row.getValue('id')}`
  },
  {
    accessorKey: 'date',
    header: 'Дата публікації',
    cell: ({ row }) => {
      const dateValue = row.getValue('date');
      return dateValue ? new Date(dateValue).toLocaleString('uk-UA', {
        day: 'numeric',
        month: 'short',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        hour12: false
      }) : 'Немає';
    }
  },
  {
    accessorKey: 'name',
    header: 'Автор'
  },
  {
    accessorKey: 'category',
    header: 'Категорія'
  },
  {
    accessorKey: 'title',
    header: 'Заголовок',
    cell: ({ row }) => {
      const postId = row.getValue('id');
      const postTitle = row.getValue('title');
      return h(NuxtLink, {
        to: `/admin/blog/posts/${postId}/edit`,
        class: 'post-title-link'
      }, () => postTitle);
    }
  },
  {
    accessorKey: 'is_published',
    header: 'Статус',
    cell: ({ row }) => {
      const isPublished = row.getValue('is_published');
      return h('div', {
        innerHTML: `<span class="status-badge ${isPublished ? 'status-published' : 'status-unpublished'}">${isPublished ? 'Опубліковано' : 'Чернетка'}</span>`
      });
    }
  }
];

const pagination = ref({
  pageIndex: 0,
  pageSize: 10
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

.table-actions {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 25px;
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

.content-wrapper {
  max-width: 1200px;
  width: 100%;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 40px;
}

.card-header {
  text-align: center;
  margin-bottom: 30px;
}

.card-header h1  {
  font-family: 'Montserrat', sans-serif;
  color: #2c3e50;
  text-align: center;
  margin-bottom: 40px;
  font-size: 3em;
  font-weight: 700;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
}

.header-subtitle {
  font-family: 'Open Sans', sans-serif;
  font-size: 1.1em;
  color: #555;
  line-height: 1.6;
  margin-top: 0;
}

.table-card {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.05);
  overflow: hidden;
  border: 1px solid #e0e0e0;
}

.table-wrapper {
  overflow-x: auto;
  padding: 0 20px;
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.loading-state, .error-state, .empty-state {
  font-family: 'Open Sans', sans-serif;
  font-size: 1.2em;
  color: #6a6a6a;
  text-align: center;
  padding: 20px;
}

.error-state {
  color: #dc3545;
}

.styled-table {
  width: 100%;
}

.styled-table :deep(thead) {
  background-color: #f8f9fa;
  border-bottom: 2px solid #e0e0e0;
}

.styled-table :deep(th) {
  font-family: 'Open Sans', sans-serif;
  text-align: left;
  padding: 15px 20px;
  font-size: 0.95em;
  font-weight: 700;
  color: #555;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.styled-table :deep(tbody tr) {
  border-bottom: 1px solid #f0f0f0;
  transition: background-color 0.2s ease;
}

.styled-table :deep(tbody tr:last-child) {
  border-bottom: none;
}

.styled-table :deep(tbody tr:hover) {
  background-color: #f9fafc;
}

.styled-table :deep(tbody tr.unpublished-row) {
  background-color: #fff9e6;
  color: #6a6a6a;
}

.styled-table :deep(tbody tr.unpublished-row:hover) {
  background-color: #fff3d9;
}

.styled-table :deep(td) {
  font-family: 'Open Sans', sans-serif;
  padding: 15px 20px;
  font-size: 0.9em;
  color: #444;
  vertical-align: middle;
}

.post-title-link {
  color: #3498db;
  text-decoration: none;
  font-weight: 600;
  transition: color 0.3s ease;
}

.post-title-link:hover {
  color: #2980b9;
  text-decoration: underline;
}

.styled-table :deep(tbody tr.unpublished-row) .post-title-link {
  color: #6a6a6a;
}

.styled-table :deep(tbody tr.unpublished-row) .post-title-link:hover {
  color: #5a5a5a;
}

:deep(.status-badge) {
  padding: 6px 12px !important;
  border-radius: 20px !important;
  font-size: 0.8em !important;
  font-weight: 600 !important;
  white-space: nowrap !important;
  display: inline-block !important;
  text-align: center !important;
  min-width: 80px !important;
}

:deep(.status-published) {
  background-color: #d4edda !important;
  color: #155724 !important;
  border: 1px solid #c3e6cb !important;
}

:deep(.status-unpublished) {
  background-color: #f8d7da !important;
  color: #721c24 !important;
  border: 1px solid #f5c6cb !important;
}

.footer-pagination {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-top: 1px solid #e0e0e0;
  background-color: #f8f9fa;
  min-height: 65px;
  flex-wrap: nowrap;
  border-bottom-left-radius: 8px;
  border-bottom-right-radius: 8px;
}

.pagination-control {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  gap: 20px;
}

.pagination-label {
  font-family: 'Open Sans', sans-serif;
  font-size: 0.9em;
  color: #6a6a6a;
  white-space: nowrap;
  flex-shrink: 0;
}

.custom-pagination :deep(nav) {
  display: flex;
  gap: 4px;
  align-items: center;
}

.custom-pagination :deep(button) {
  background-color: white !important;
  border: 1px solid #dee2e6 !important;
  padding: 8px 12px !important;
  border-radius: 6px !important;
  font-size: 0.875rem !important;
  color: #495057 !important;
  cursor: pointer !important;
  transition: all 0.15s ease-in-out !important;
  min-width: 40px !important;
  height: 40px !important;
  display: flex !important;
  align-items: center !important;
  justify-content: center !important;
  font-weight: 500 !important;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05) !important;
}

.custom-pagination :deep(button:hover:not(:disabled)) {
  background-color: #e9ecef !important;
  border-color: #adb5bd !important;
  color: #495057 !important;
  transform: translateY(-1px) !important;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1) !important;
}

.custom-pagination :deep(button[aria-current="page"]) {
  background-color: #007bff !important;
  border-color: #007bff !important;
  color: white !important;
  font-weight: 600 !important;
  box-shadow: 0 2px 4px rgba(0, 123, 255, 0.25) !important;
}

.custom-pagination :deep(button:disabled) {
  opacity: 0.6 !important;
  cursor: not-allowed !important;
  background-color: #f8f9fa !important;
  border-color: #dee2e6 !important;
  color: #6c757d !important;
  transform: none !important;
  box-shadow: none !important;
}

.custom-pagination :deep(button:focus) {
  outline: 2px solid #007bff !important;
  outline-offset: 2px !important;
}
</style>