<template>
  <div class="flex flex-col items-center justify-center min-h-screen p-8 bg-gray-100 relative">
    <NuxtLink
        to="/"
        class="absolute top-8 left-8 inline-flex items-center px-5 py-2.5 bg-gray-600 text-white font-semibold rounded-lg shadow-md transition-all duration-300 ease-in-out hover:bg-gray-700 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-opacity-75"
    >
      На головну
    </NuxtLink>

    <div class="w-full max-w-screen-xl bg-white rounded-xl shadow-lg p-10">
      <div class="text-center mb-10">
        <h1 class="font-['Montserrat'] text-5xl font-extrabold text-gray-800 mb-10 drop-shadow-md">Список постів (NuxtUI Таблиця)</h1>
        <div class="flex justify-end mb-6">
          <NuxtLink
              to="/admin/blog/posts/create"
              class="inline-flex items-center px-6 py-3 bg-blue-500 text-white font-semibold rounded-lg shadow-md transition-all duration-300 ease-in-out hover:bg-blue-600 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-75"
          >
            Додати новий пост
          </NuxtLink>
        </div>
      </div>

      <div class="bg-white rounded-lg shadow-md border border-gray-200 overflow-hidden">
        <div class="overflow-x-auto min-h-52 flex items-center justify-center p-5">
          <template v-if="pending">
            <div class="text-lg text-gray-600 font-medium">Завантаження постів...</div>
          </template>
          <template v-else-if="error">
            <div class="text-lg text-red-600 font-medium">Помилка завантаження постів: {{ error.message }}</div>
          </template>
          <template v-else-if="posts.length === 0">
            <div class="text-lg text-gray-600 font-medium">Постів поки що немає.</div>
          </template>
          <template v-else>
            <UTable
                ref="table"
                v-model:pagination="pagination"
                :data="posts"
                :columns="columns"
                :pagination-options="{ getPaginationRowModel: getPaginationRowModel() }"
                class="w-full table-fixed"
                :thead-class="'bg-gray-50 border-b-2 border-gray-200'"
                :tbody-tr-class="(row) => getRowClass(row)"
                :tbody-td-class="'py-4 px-5 text-gray-700 text-sm'"
                :th-class="'py-4 px-5 text-left text-sm font-bold text-gray-600 uppercase tracking-wider'"
            />
          </template>
        </div>

        <div v-if="!pending && !error && posts.length > 0" class="flex items-center justify-between px-5 py-4 bg-gray-50 border-t border-gray-200 rounded-b-lg flex-wrap gap-x-5">
          <div class="text-sm text-gray-600 whitespace-nowrap">
            Показано {{ ((pagination.pageIndex) * pagination.pageSize) + 1 }} -
            {{ Math.min((pagination.pageIndex + 1) * pagination.pageSize, posts.length) }}
            з {{ posts.length }} результатів
          </div>
          <UPagination
              :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
              :items-per-page="table?.tableApi?.getState().pagination.pageSize"
              :total="table?.tableApi?.getFilteredRowModel().rows.length"
              @update:page="p => table?.tableApi?.setPageIndex(p - 1)"
              class="flex gap-1"
              :ui="{
              wrapper: 'flex items-center gap-1',
              base: 'flex-1 min-w-0',
              item: {
                base: 'w-10 h-10 flex items-center justify-center rounded-md text-sm font-medium transition-all duration-150',
                active: 'bg-blue-600 text-white shadow-md',
                inactive: 'bg-white text-gray-700 border border-gray-300 hover:bg-gray-100',
                size: 'text-sm'
              },
              prev: { base: 'w-10 h-10', icon: 'i-heroicons-chevron-left' },
              next: { base: 'w-10 h-10', icon: 'i-heroicons-chevron-right' }
            }"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, h } from 'vue';
import { getPaginationRowModel } from '@tanstack/vue-table';
import type { TableColumn } from '@nuxt/ui';

const table = ref();

type Post = {
  id: string;
  name: string;
  category: string;
  title: string;
  date: string;
  is_published: boolean;
  slug: string;
};

const posts = ref<Post[]>([]);
const config = useRuntimeConfig();

const getRowClass = (row: any) => {
  const isPublished = row.original.is_published;
  return isPublished ? 'bg-white hover:bg-blue-50 hover:shadow-sm transform transition-all duration-200 ease-in-out cursor-pointer' : 'bg-amber-50 text-gray-600 hover:bg-amber-100 hover:shadow-sm transform transition-all duration-200 ease-in-out cursor-pointer';
};

const handleRowClick = async (post: Post) => {
  try {
    console.log('Navigating to:', `/blog/${post.slug}?source=nuxt-ui`);
    await navigateTo(`/blog/${post.slug}?source=nuxt-ui`);
  } catch (error) {
    console.error('Navigation error:', error);
    window.location.href = `/blog/${post.slug}?source=nuxt-ui`;
  }
};

const handleDeleteClick = async (post: Post) => {
  const confirmed = confirm(`Ви впевнені, що хочете видалити пост "${post.title}"? Цю дію неможливо скасувати.`);

  if (!confirmed) return;

  try {
    await $fetch(`${config.public.apiBase}/blog/posts/${post.slug}`, {
      method: 'DELETE'
    });

    posts.value = posts.value.filter(p => p.slug !== post.slug);

    alert('Пост успішно видалено!');

  } catch (error) {
    console.error('Delete error:', error);
    alert('Помилка при видаленні поста');
  }
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
          is_published: post.is_published,
          slug: post.slug
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
    cell: ({ row }) => h('div', {
      onClick: () => handleRowClick(row.original),
      class: 'cursor-pointer w-full h-full flex items-center'
    }, `#${row.getValue('id')}`)
  },
  {
    accessorKey: 'name',
    header: 'Автор',
    cell: ({ row }) => h('div', {
      onClick: () => handleRowClick(row.original),
      class: 'cursor-pointer w-full h-full flex items-center'
    }, row.getValue('name'))
  },
  {
    accessorKey: 'category',
    header: 'Категорія',
    cell: ({ row }) => h('div', {
      onClick: () => handleRowClick(row.original),
      class: 'cursor-pointer w-full h-full flex items-center'
    }, row.getValue('category'))
  },
  {
    accessorKey: 'title',
    header: 'Заголовок',
    cell: ({ row }) => {
      const postTitle = row.getValue('title');
      return h('div', {
        class: 'cursor-pointer text-gray-800 font-medium hover:text-gray-900 transition-colors duration-300 w-full h-full flex items-center break-words',
        onClick: () => handleRowClick(row.original)
      }, postTitle);
    }
  },
  {
    accessorKey: 'date',
    header: 'Дата публікації',
    cell: ({ row }) => {
      const dateValue = row.getValue('date');
      const formattedDate = dateValue ? new Date(dateValue).toLocaleString('uk-UA', {
        day: 'numeric',
        month: 'short',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        hour12: false
      }) : 'Немає';
      return h('div', {
        onClick: () => handleRowClick(row.original),
        class: 'cursor-pointer w-full h-full flex items-center'
      }, formattedDate);
    }
  },
  {
    accessorKey: 'is_published',
    header: 'Статус',
    cell: ({ row }) => {
      const isPublished = row.getValue('is_published');
      return h('div', {
        onClick: () => handleRowClick(row.original),
        class: 'cursor-pointer w-full h-full flex items-center',
        innerHTML: `<span class="inline-block px-3 py-1 rounded-full text-xs font-semibold whitespace-nowrap text-center min-w-[80px] ${isPublished ? 'bg-green-100 text-green-800 border border-green-200' : 'bg-red-100 text-red-800 border border-red-200'}">${isPublished ? 'Опубліковано' : 'Чернетка'}</span>`
      });
    }
  },
  {
    accessorKey: 'actions',
    header: 'Дії',
    cell: ({ row }) => {
      const postSlug = row.original.slug;

      return h('div', {
        class: 'flex justify-center items-center gap-2'
      }, [
        h('button', {
          class: 'px-3 py-1.5 bg-blue-500 text-white border border-blue-500 rounded-md text-sm font-medium cursor-pointer transition-all duration-300 ease-in-out hover:bg-blue-600 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-75',
          onClick: async (e: Event) => {
            e.stopPropagation();
            try {
              await navigateTo(`/admin/blog/posts/${postSlug}/edit`);
            } catch (error) {
              console.error('Edit navigation error:', error);
              window.location.href = `/admin/blog/posts/${postSlug}/edit`;
            }
          }
        }, 'Редагувати'),
        h('button', {
          class: 'px-3 py-1.5 bg-red-500 text-white border border-red-500 rounded-md text-sm font-medium cursor-pointer transition-all duration-300 ease-in-out hover:bg-red-600 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-75',
          onClick: (e: Event) => {
            e.stopPropagation();
            handleDeleteClick(row.original);
          }
        }, 'Видалити')
      ]);
    }
  }
];

const pagination = ref({
  pageIndex: 0,
  pageSize: 10
});
</script>

<style scoped>
.table-fixed :deep(th),
.table-fixed :deep(td) {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: normal;
}

.table-fixed :deep(td > .break-words) {
  white-space: normal;
  word-break: break-word;
}
</style>