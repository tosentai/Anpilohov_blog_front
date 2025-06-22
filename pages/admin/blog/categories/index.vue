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
        <h1 class="font-['Montserrat'] text-5xl font-extrabold text-gray-800 mb-10 drop-shadow-md">Список категорій (NuxtUI Таблиця)</h1>
        <div class="flex justify-between items-center mb-6 gap-4">
          <div class="flex-1 max-w-md">
            <div class="relative">
              <input
                  v-model="searchQuery"
                  type="text"
                  placeholder="Пошук по всім полям..."
                  class="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-all duration-200 text-sm"
                  @input="handleSearch"
              >
              <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                <svg class="h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
                </svg>
              </div>
              <div v-if="searchQuery" class="absolute inset-y-0 right-0 pr-3 flex items-center">
                <button
                    @click="clearSearch"
                    class="text-gray-400 hover:text-gray-600 transition-colors"
                >
                  <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                  </svg>
                </button>
              </div>
            </div>
          </div>

          <NuxtLink
              to="/admin/blog/categories/create"
              class="inline-flex items-center px-6 py-3 bg-blue-500 text-white font-semibold rounded-lg shadow-md transition-all duration-300 ease-in-out hover:bg-blue-600 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-75 whitespace-nowrap"
          >
            Додати нову категорію
          </NuxtLink>
        </div>

        <div v-if="searchQuery && !pending" class="text-sm text-gray-600 mb-4 text-left">
          Знайдено {{ filteredCategories.length }} з {{ categories.length }} категорій
          <span v-if="searchQuery" class="font-medium">для запиту "{{ searchQuery }}"</span>
        </div>
      </div>

      <div class="bg-white rounded-lg shadow-md border border-gray-200 overflow-hidden">
        <div class="overflow-x-auto min-h-52 flex items-center justify-center p-5">
          <template v-if="pending">
            <div class="text-lg text-gray-600 font-medium">Завантаження категорій...</div>
          </template>
          <template v-else-if="error">
            <div class="text-lg text-red-600 font-medium">Помилка завантаження категорій: {{ error.message }}</div>
          </template>
          <template v-else-if="filteredCategories.length === 0 && !searchQuery">
            <div class="text-lg text-gray-600 font-medium">Категорій поки що немає.</div>
          </template>
          <template v-else-if="filteredCategories.length === 0 && searchQuery">
            <div class="text-center">
              <div class="text-lg text-gray-600 font-medium mb-2">Нічого не знайдено</div>
              <div class="text-sm text-gray-500">Спробуйте змінити пошуковий запит</div>
              <button
                  @click="clearSearch"
                  class="mt-3 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 transition-colors text-sm"
              >
                Очистити пошук
              </button>
            </div>
          </template>
          <template v-else>
            <UTable
                ref="table"
                v-model:pagination="pagination"
                :data="filteredCategories"
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

        <div v-if="!pending && !error && filteredCategories.length > 0" class="flex items-center justify-between px-5 py-4 bg-gray-50 border-t border-gray-200 rounded-b-lg flex-wrap gap-x-5">
          <div class="text-sm text-gray-600 whitespace-nowrap">
            Показано {{ ((pagination.pageIndex) * pagination.pageSize) + 1 }} -
            {{ Math.min((pagination.pageIndex + 1) * pagination.pageSize, filteredCategories.length) }}
            з {{ filteredCategories.length }} результатів
            <span v-if="searchQuery">(відфільтровано з {{ categories.length }})</span>
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
import { ref, h, computed, watch } from 'vue';
import { getPaginationRowModel } from '@tanstack/vue-table';
import type { TableColumn } from '@nuxt/ui';

const table = ref();

type Category = {
  id: string;
  title: string;
  slug: string;
  description: string | null;
  parent_id: string | null;
  created_at: string;
  updated_at: string;
};

const categories = ref<Category[]>([]);
const searchQuery = ref('');
const config = useRuntimeConfig();

const filteredCategories = computed(() => {
  if (!searchQuery.value.trim()) {
    return categories.value;
  }

  const query = searchQuery.value.toLowerCase().trim();

  return categories.value.filter(category => {
    const searchFields = [
      category.id,
      category.title,
      category.slug,
      category.description || '',
      category.parent_id || '',
      new Date(category.created_at).toLocaleString('uk-UA'),
      new Date(category.updated_at).toLocaleString('uk-UA')
    ];

    return searchFields.some(field =>
        String(field).toLowerCase().includes(query)
    );
  });
});

const handleSearch = () => {
  pagination.value.pageIndex = 0;
};

const clearSearch = () => {
  searchQuery.value = '';
  pagination.value.pageIndex = 0;
};

watch(filteredCategories, () => {
  if (pagination.value.pageIndex > 0) {
    const maxPage = Math.ceil(filteredCategories.value.length / pagination.value.pageSize) - 1;
    if (pagination.value.pageIndex > maxPage) {
      pagination.value.pageIndex = Math.max(0, maxPage);
    }
  }
});

const getRowClass = (row: any) => {
  return 'bg-white hover:bg-blue-50 hover:shadow-sm transform transition-all duration-200 ease-in-out cursor-pointer';
};

const handleRowClick = async (category: Category) => {
  try {
    console.log('Navigating to:', `/admin/blog/categories/${category.id}/edit`);
    await navigateTo(`/admin/blog/categories/${category.id}/edit`);
  } catch (error) {
    console.error('Navigation error:', error);
    window.location.href = `/admin/blog/categories/${category.id}/edit`;
  }
};

const deleteCategory = async (id: string, event: Event) => {
  event.stopPropagation();

  if (!confirm('Ви впевнені, що хочете видалити цю категорію?')) {
    return;
  }

  try {
    await $fetch(`${config.public.apiBase}/blog/categories/${id}`, {
      method: 'DELETE'
    });

    await refresh();
  } catch (error) {
    console.error('Помилка видалення категорії:', error);
    alert('Не вдалося видалити категорію.');
  }
};

const { data, pending, error, refresh } = await useAsyncData(
    'all-blog-categories',
    async () => {
      let allCategories: Category[] = [];
      let currentPage = 1;
      let lastPage = 1;

      do {
        const response: any = await $fetch(`${config.public.apiBase}/blog/categories?page=${currentPage}`);
        const pageCategories = response.data.map((category: any) => ({
          id: String(category.id),
          title: category.title,
          slug: category.slug,
          description: category.description,
          parent_id: category.parent_id ? String(category.parent_id) : null,
          created_at: category.created_at,
          updated_at: category.updated_at
        })) as Category[];

        allCategories = allCategories.concat(pageCategories);
        lastPage = response.meta.last_page;
        currentPage++;
      } while (currentPage <= lastPage);

      return allCategories;
    }
);

if (data.value) {
  categories.value = data.value;
}

const columns: TableColumn<Category>[] = [
  {
    accessorKey: 'id',
    header: '#',
    cell: ({ row }) => h('div', {
      onClick: () => handleRowClick(row.original),
      class: 'cursor-pointer w-full h-full flex items-center'
    }, `#${row.getValue('id')}`)
  },
  {
    accessorKey: 'title',
    header: 'Назва',
    cell: ({ row }) => {
      const categoryTitle = row.getValue('title');
      return h('div', {
        class: 'cursor-pointer text-gray-800 font-medium hover:text-gray-900 transition-colors duration-300 w-full h-full flex items-center break-words',
        onClick: () => handleRowClick(row.original)
      }, categoryTitle);
    }
  },
  {
    accessorKey: 'slug',
    header: 'Slug',
    cell: ({ row }) => h('div', {
      onClick: () => handleRowClick(row.original),
      class: 'cursor-pointer w-full h-full flex items-center'
    }, row.getValue('slug'))
  },
  {
    accessorKey: 'description',
    header: 'Опис',
    cell: ({ row }) => {
      const description = row.getValue('description');
      const truncatedDescription = description
          ? (String(description).length > 50 ? String(description).substring(0, 50) + '...' : String(description))
          : '-';
      return h('div', {
        onClick: () => handleRowClick(row.original),
        class: 'cursor-pointer w-full h-full flex items-center break-words'
      }, truncatedDescription);
    }
  },
  {
    accessorKey: 'parent_id',
    header: 'Батьківська категорія',
    cell: ({ row }) => {
      const parentId = row.getValue('parent_id');
      return h('div', {
        onClick: () => handleRowClick(row.original),
        class: 'cursor-pointer w-full h-full flex items-center'
      }, parentId ? `#${parentId}` : '-');
    }
  },
  {
    accessorKey: 'created_at',
    header: 'Дата створення',
    cell: ({ row }) => {
      const dateValue = row.getValue('created_at');
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
    accessorKey: 'actions',
    header: 'Дії',
    cell: ({ row }) => {
      const categoryId = row.getValue('id');
      return h('div', {
        class: 'flex justify-center items-center gap-2'
      }, [
        h('button', {
          class: 'px-3 py-1.5 bg-blue-500 text-white border border-blue-500 rounded-md text-sm font-medium cursor-pointer transition-all duration-300 ease-in-out hover:bg-blue-600 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-75',
          onClick: async (e: Event) => {
            e.stopPropagation();
            try {
              await navigateTo(`/admin/blog/categories/${categoryId}/edit`);
            } catch (error) {
              console.error('Edit navigation error:', error);
              window.location.href = `/admin/blog/categories/${categoryId}/edit`;
            }
          }
        }, 'Редагувати'),
        h('button', {
          class: 'px-3 py-1.5 bg-red-500 text-white border border-red-500 rounded-md text-sm font-medium cursor-pointer transition-all duration-300 ease-in-out hover:bg-red-600 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-75',
          onClick: (e: Event) => deleteCategory(String(categoryId), e)
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