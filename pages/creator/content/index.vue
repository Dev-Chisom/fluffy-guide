<template>
  <div>
    <Head>
      <Title>Content Management - Whispers</Title>
    </Head>

    <div class="sm:flex sm:items-center sm:justify-between">
      <div>
        <h1 class="text-2xl font-bold text-gray-900">Content Management</h1>
        <p class="mt-1 text-sm text-gray-500">
          Create, edit, and manage your content.
        </p>
      </div>
      <div class="mt-4 sm:mt-0">
        <NuxtLink to="/creator/content/new" class="btn-primary">
          <Icon name="lucide:plus" class="mr-2 h-4 w-4" />
          Create New Post
        </NuxtLink>
      </div>
    </div>

    <!-- Filters -->
    <div class="mt-6 bg-white rounded-lg shadow-sm p-4 sm:flex sm:items-center space-y-4 sm:space-y-0 sm:space-x-4">
      <div class="flex-1">
        <label for="search" class="sr-only">Search</label>
        <div class="relative rounded-md shadow-sm">
          <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
            <Icon name="lucide:search" class="h-5 w-5 text-gray-400" />
          </div>
          <input
            id="search"
            v-model="search"
            type="search"
            placeholder="Search content..."
            class="form-input pl-10"
          />
        </div>
      </div>
      <div class="w-full sm:w-auto">
        <select
          v-model="visibilityFilter"
          class="form-input"
        >
          <option value="">All Visibility</option>
          <option value="public">Public</option>
          <option value="subscribers">Subscribers Only</option>
          <option value="ppv">Pay-per-view</option>
        </select>
      </div>
      <div class="w-full sm:w-auto">
        <select
          v-model="sortBy"
          class="form-input"
        >
          <option value="newest">Newest First</option>
          <option value="oldest">Oldest First</option>
          <option value="popular">Most Popular</option>
        </select>
      </div>
    </div>

    <!-- Content table/list -->
    <div class="mt-6 bg-white shadow-sm rounded-lg overflow-hidden">
      <div class="bg-gray-50 px-4 py-3 border-b border-gray-200 text-xs font-medium text-gray-500 uppercase tracking-wider">
        Content List
      </div>

      <div v-if="loading" class="p-8 text-center">
        <Icon name="lucide:loader" class="h-8 w-8 mx-auto animate-spin text-primary-500" />
        <p class="mt-2 text-gray-500">Loading content...</p>
      </div>

      <div v-else-if="filteredPosts.length === 0" class="p-8 text-center">
        <Icon name="lucide:file-x" class="h-12 w-12 mx-auto text-gray-400" />
        <h3 class="mt-2 text-sm font-medium text-gray-900">No content found</h3>
        <p class="mt-1 text-sm text-gray-500">
          {{ search ? 'Try adjusting your search or filters.' : 'Get started by creating your first post.' }}
        </p>
        <div class="mt-6">
          <NuxtLink to="/creator/content/new" class="btn-primary">
            <Icon name="lucide:plus" class="mr-2 h-4 w-4" />
            Create New Post
          </NuxtLink>
        </div>
      </div>

      <ul v-else class="divide-y divide-gray-200">
        <li v-for="post in filteredPosts" :key="post.id" class="hover:bg-gray-50">
          <div class="px-4 py-4 sm:px-6 flex items-center">
            <div class="w-16 h-16 flex-shrink-0 rounded overflow-hidden bg-gray-100 mr-4">
              <img
                v-if="post.mediaUrls && post.mediaUrls.length > 0"
                :src="post.mediaUrls[0]"
                :alt="post.title"
                class="w-full h-full object-cover"
              />
              <div v-else class="w-full h-full flex items-center justify-center text-gray-400">
                <Icon name="lucide:file" class="h-6 w-6" />
              </div>
            </div>
            
            <div class="min-w-0 flex-1">
              <div class="flex items-center">
                <h3 class="text-sm font-medium text-gray-900 truncate">{{ post.title }}</h3>
                <span 
                  v-if="post.visibility === 'subscribers'" 
                  class="ml-2 badge badge-primary"
                >
                  Subscribers
                </span>
                <span 
                  v-else-if="post.visibility === 'ppv'" 
                  class="ml-2 badge badge-secondary"
                >
                  Pay-per-view
                </span>
                <span 
                  v-else 
                  class="ml-2 badge badge-success"
                >
                  Public
                </span>
                <span 
                  v-if="post.scheduledFor" 
                  class="ml-2 badge badge-warning"
                >
                  Scheduled
                </span>
              </div>
              
              <div class="mt-1">
                <p class="text-sm text-gray-500 truncate">
                  {{ truncateContent(post.content) }}
                </p>
              </div>
              
              <div class="mt-2 flex items-center text-xs text-gray-500 space-x-4">
                <span class="flex items-center">
                  <Icon name="lucide:calendar" class="h-4 w-4 mr-1" />
                  {{ formatDate(post.createdAt) }}
                </span>
                <span class="flex items-center">
                  <Icon name="lucide:heart" class="h-4 w-4 mr-1" />
                  {{ post.likes }}
                </span>
                <span class="flex items-center">
                  <Icon name="lucide:message-square" class="h-4 w-4 mr-1" />
                  {{ post.comments }}
                </span>
                <span v-if="post.price" class="flex items-center">
                  <Icon name="lucide:dollar-sign" class="h-4 w-4 mr-1" />
                  {{ post.price }}
                </span>
              </div>
            </div>
            
            <div class="ml-4 flex-shrink-0 flex space-x-2">
              <button
                class="p-2 text-gray-500 hover:text-primary-600 rounded-md hover:bg-gray-100"
                @click="editPost(post.id)"
              >
                <Icon name="lucide:pencil" class="h-5 w-5" />
                <span class="sr-only">Edit</span>
              </button>
              <button
                class="p-2 text-gray-500 hover:text-error-600 rounded-md hover:bg-gray-100"
                @click="confirmDelete(post.id)"
              >
                <Icon name="lucide:trash-2" class="h-5 w-5" />
                <span class="sr-only">Delete</span>
              </button>
            </div>
          </div>
        </li>
      </ul>
    </div>

    <!-- Delete confirmation modal -->
    <div v-if="showDeleteModal" class="fixed inset-0 z-50 overflow-y-auto">
      <div class="flex items-center justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0">
        <div class="fixed inset-0 transition-opacity">
          <div class="absolute inset-0 bg-gray-500 opacity-75"></div>
        </div>
        <span class="hidden sm:inline-block sm:align-middle sm:h-screen">&#8203;</span>
        <div class="inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg sm:w-full">
          <div class="bg-white px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
            <div class="sm:flex sm:items-start">
              <div class="mx-auto flex-shrink-0 flex items-center justify-center h-12 w-12 rounded-full bg-error-100 sm:mx-0 sm:h-10 sm:w-10">
                <Icon name="lucide:alert-triangle" class="h-6 w-6 text-error-600" />
              </div>
              <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left">
                <h3 class="text-lg leading-6 font-medium text-gray-900">
                  Delete Content
                </h3>
                <div class="mt-2">
                  <p class="text-sm text-gray-500">
                    Are you sure you want to delete this content? This action cannot be undone.
                  </p>
                </div>
              </div>
            </div>
          </div>
          <div class="bg-gray-50 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse">
            <button
              @click="deletePost"
              class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-error-600 text-base font-medium text-white hover:bg-error-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-error-500 sm:ml-3 sm:w-auto sm:text-sm"
            >
              Delete
            </button>
            <button
              @click="showDeleteModal = false"
              class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-primary-500 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm"
            >
              Cancel
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useContentStore } from '~/stores/content';

definePageMeta({
  layout: 'creator',
  middleware: ['auth'],
  meta: {
    requiresAuth: true,
    requiresCreator: true
  }
});

const contentStore = useContentStore();
const toast = inject('toast');

// State
const loading = ref(true);
const search = ref('');
const visibilityFilter = ref('');
const sortBy = ref('newest');
const showDeleteModal = ref(false);
const postToDelete = ref(null);

// Fetch posts on mount
onMounted(async () => {
  try {
    await contentStore.fetchPosts({ creatorId: '123' }); // In a real app, this would be the current user's ID
    loading.value = false;
  } catch (error) {
    toast.error('Failed to load content. Please try again.');
    loading.value = false;
  }
});

// Computed properties
const filteredPosts = computed(() => {
  let result = [...contentStore.posts];
  
  // Apply search filter
  if (search.value) {
    const searchLower = search.value.toLowerCase();
    result = result.filter(post => 
      post.title.toLowerCase().includes(searchLower) || 
      post.content.toLowerCase().includes(searchLower)
    );
  }
  
  // Apply visibility filter
  if (visibilityFilter.value) {
    result = result.filter(post => post.visibility === visibilityFilter.value);
  }
  
  // Apply sorting
  switch (sortBy.value) {
    case 'newest':
      result.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      break;
    case 'oldest':
      result.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt));
      break;
    case 'popular':
      result.sort((a, b) => b.likes - a.likes);
      break;
  }
  
  return result;
});

// Methods
function truncateContent(content, length = 80) {
  if (content.length <= length) return content;
  return content.substring(0, length) + '...';
}

function formatDate(dateString) {
  const date = new Date(dateString);
  return date.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
  });
}

function editPost(id) {
  navigateTo(`/creator/content/edit/${id}`);
}

function confirmDelete(id) {
  postToDelete.value = id;
  showDeleteModal.value = true;
}

async function deletePost() {
  if (!postToDelete.value) return;
  
  try {
    await contentStore.deletePost(postToDelete.value);
    toast.success('Content deleted successfully');
    showDeleteModal.value = false;
    postToDelete.value = null;
  } catch (error) {
    toast.error('Failed to delete content. Please try again.');
  }
}
</script>