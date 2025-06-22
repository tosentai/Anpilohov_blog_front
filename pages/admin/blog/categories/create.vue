<template>
  <div class="flex flex-col items-center justify-center min-h-screen p-8 bg-gray-100 relative">
    <NuxtLink
        to="/admin/blog/categories"
        class="absolute top-8 left-8 inline-flex items-center px-5 py-2.5 bg-gray-600 text-white font-semibold rounded-lg shadow-md transition-all duration-300 ease-in-out hover:bg-gray-700 hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-opacity-75"
    >
      ← Назад до списку
    </NuxtLink>

    <div class="w-full max-w-2xl bg-white rounded-xl shadow-lg p-10">
      <div class="text-center mb-10">
        <h1 class="font-['Montserrat'] text-5xl font-extrabold text-gray-800 mb-4 drop-shadow-md">Створити категорію</h1>
        <p class="text-lg text-gray-600">Заповніть форму для додавання нової категорії блогу</p>
      </div>

      <div class="bg-white rounded-lg shadow-md border border-gray-200 overflow-hidden">
        <div class="p-8">
          <UForm :schema="schema" :state="state" class="flex flex-col gap-6" @submit="onSubmit">
            <UFormGroup label="Назва категорії" name="title" :error="errors.title" help="Мінімум 3 символи, максимум 200">
              <UInput
                  v-model="state.title"
                  class="w-full"
                  :class="{ 'border-red-500': errors.title }"
                  placeholder="Введіть назву категорії"
              />
            </UFormGroup>

            <UFormGroup label="Slug (Псевдонім)" name="slug" :error="errors.slug" help="Залиште порожнім для автоматичної генерації">
              <UInput
                  v-model="state.slug"
                  class="w-full"
                  :class="{ 'border-red-500': errors.slug }"
                  placeholder="category-slug"
              />
            </UFormGroup>

            <UFormGroup label="Опис" name="description" :error="errors.description" help="Максимум 500 символів">
              <UTextarea
                  v-model="state.description"
                  :rows="4"
                  class="w-full"
                  :class="{ 'border-red-500': errors.description }"
                  placeholder="Опишіть категорію..."
              />
            </UFormGroup>

            <UFormGroup label="Батьківська категорія" name="parent_id" :error="errors.parent_id">
              <!-- Звичайний HTML select -->
              <select
                  v-model="state.parent_id"
                  class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 bg-white"
                  :class="{ 'border-red-500 focus:ring-red-500 focus:border-red-500': errors.parent_id }"
                  :disabled="isLoadingCategories"
              >
                <option value="">Оберіть батьківську категорію</option>
                <option :value="null">Без батьківської категорії</option>
                <option
                    v-for="category in availableCategories"
                    :key="category.id"
                    :value="category.id"
                >
                  {{ category.title }}
                </option>
              </select>

              <div class="text-xs text-gray-500 mt-1">
                Доступно категорій: {{ availableCategories.length }}
                <span v-if="isLoadingCategories" class="text-blue-500 ml-2">
                  (Завантаження...)
                </span>
              </div>

              <!-- Debug інформація -->
              <div v-if="$config.public.NODE_ENV === 'development'" class="text-xs text-blue-500 mt-1">
                Debug: Selected value = {{ state.parent_id }}, Options loaded = {{ !isLoadingCategories }}
              </div>
            </UFormGroup>

            <div class="flex justify-end gap-3 pt-6 border-t border-gray-200 mt-6">
              <UButton
                  type="button"
                  color="gray"
                  variant="outline"
                  class="px-6 py-2.5 transition-all duration-300 ease-in-out hover:-translate-y-0.5"
                  @click="navigateTo('/admin/blog/categories')"
              >
                Відмінити
              </UButton>
              <UButton
                  type="submit"
                  :loading="isSubmitting"
                  color="primary"
                  class="px-6 py-2.5 bg-blue-500 hover:bg-blue-600 transition-all duration-300 ease-in-out hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-75"
              >
                <template v-if="isSubmitting">
                  Збереження...
                </template>
                <template v-else>
                  Зберегти категорію
                </template>
              </UButton>
            </div>
          </UForm>
        </div>
      </div>

      <!-- Loading state -->
      <div v-if="isLoadingCategories" class="text-center mt-4">
        <div class="text-gray-600">Завантаження категорій...</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import { z } from 'zod';
import { useToast } from '#imports';
import { useRuntimeConfig, navigateTo } from '#app';

interface Category {
  id: number;
  title: string;
  slug: string;
  description: string | null;
  parent_id: number | null;
  created_at: string;
  updated_at: string;
}

const schema = z.object({
  title: z.string()
      .min(3, 'Назва повинна містити мінімум 3 символи')
      .max(200, 'Назва не може перевищувати 200 символів'),
  slug: z.string()
      .max(200, 'Slug не може перевищувати 200 символів')
      .optional()
      .or(z.literal('')),
  description: z.string()
      .max(500, 'Опис не може перевищувати 500 символів')
      .optional()
      .or(z.literal('')),
  parent_id: z.number()
      .nullable()
      .optional(),
});

type CategoryForm = z.infer<typeof schema>;

const state = ref<CategoryForm>({
  title: '',
  slug: '',
  description: '',
  parent_id: null,
});

const errors = ref<{ [key: string]: string | undefined }>({});
const isSubmitting = ref(false);
const isLoadingCategories = ref(false);
const toast = useToast();
const config = useRuntimeConfig();

const availableCategories = ref<Category[]>([]);

// Computed для відображення обраної категорії (не потрібен для звичайного select)
// const selectedCategoryLabel = computed(() => {
//   if (state.value.parent_id === null) {
//     return 'Без батьківської категорії';
//   }
//   const selected = categoryOptions.value.find(opt => opt.value === state.value.parent_id);
//   return selected ? selected.label : 'Оберіть батьківську категорію';
// });

const fetchCategoriesForSelect = async () => {
  isLoadingCategories.value = true;
  try {
    console.log("Fetching categories for select...");
    const response = await $fetch<{ data: Category[] }>(`${config.public.apiBase}/blog/categories?per_page=1000`);
    console.log("Raw API response:", response);

    if (response && response.data && Array.isArray(response.data)) {
      availableCategories.value = response.data;
      console.log("Available categories:", availableCategories.value);
      console.log("Categories count:", availableCategories.value.length);
    } else {
      console.error("Invalid response structure:", response);
      availableCategories.value = [];
    }
  } catch (error) {
    console.error('Error fetching categories for select (create.vue):', error);
    availableCategories.value = [];
    toast.add({
      title: 'Помилка завантаження списку категорій',
      description: 'Не вдалося завантажити список батьківських категорій.',
      icon: 'i-heroicons-x-circle',
      color: 'red'
    });
  } finally {
    isLoadingCategories.value = false;
  }
};

const onSubmit = async () => {
  errors.value = {};

  try {
    schema.parse(state.value);
  } catch (validationError: any) {
    if (validationError instanceof z.ZodError) {
      validationError.errors.forEach(err => {
        errors.value[err.path[0]] = err.message;
      });
      toast.add({
        title: 'Помилка валідації',
        description: 'Будь ласка, перевірте введені дані.',
        icon: 'i-heroicons-exclamation-triangle',
        color: 'yellow'
      });
      return;
    }
  }

  isSubmitting.value = true;
  try {
    const result = await $fetch(`${config.public.apiBase}/blog/categories`, {
      method: 'POST',
      body: state.value,
    });
    console.log("Category creation response (create.vue):", result);
    toast.add({
      title: 'Категорію створено',
      description: 'Категорія була успішно додана.',
      icon: 'i-heroicons-check-circle',
      color: 'green'
    });
    navigateTo('/admin/blog/categories');
  } catch (error: any) {
    console.error('Error creating category (create.vue):', error);
    if (error.statusCode === 422 && error.data && error.data.errors) {
      for (const key in error.data.errors) {
        errors.value[key] = error.data.errors[key][0];
      }
      toast.add({
        title: 'Помилка валідації на сервері',
        description: 'Перевірте введені дані та спробуйте ще раз.',
        icon: 'i-heroicons-exclamation-triangle',
        color: 'yellow'
      });
    } else {
      toast.add({
        title: 'Помилка',
        description: 'Виникла помилка під час створення категорії.',
        icon: 'i-heroicons-x-circle',
        color: 'red'
      });
    }
  } finally {
    isSubmitting.value = false;
  }
};

onMounted(() => {
  fetchCategoriesForSelect();
});
</script>

<style scoped>
</style>