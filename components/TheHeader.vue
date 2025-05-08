<template>
  <header class="bg-white shadow-sm">
    <div class="container mx-auto px-4 sm:px-6 lg:px-8">
      <div class="flex items-center justify-between h-16">
        <!-- Logo -->
        <div class="flex items-center flex-shrink-0">
          <NuxtLink to="/" class="flex items-center">
            <img
              class="h-8 w-auto sm:h-10"
              src="/logo.svg"
              alt="Whispers"
            />
            <span class="ml-2 text-xl font-bold text-primary-600 hidden sm:block">Whispers</span>
          </NuxtLink>
        </div>

        <!-- Desktop navigation -->
        <nav class="hidden md:flex items-center space-x-4">
          <NuxtLink
            v-for="item in navItems"
            :key="item.name"
            :to="item.href"
            :class="[
              'px-3 py-2 rounded-md text-sm font-medium',
              $route.path === item.href
                ? 'text-primary-700 bg-primary-50'
                : 'text-gray-700 hover:text-primary-600 hover:bg-gray-50'
            ]"
          >
            {{ item.name }}
          </NuxtLink>
        </nav>

        <!-- Right section -->
        <div class="flex items-center space-x-3">
          <!-- Search -->
          <button
            type="button"
            class="p-2 rounded-full text-gray-500 hover:text-gray-700 hover:bg-gray-100"
            @click="openSearch"
          >
            <Icon name="lucide:search" class="w-5 h-5" />
          </button>

          <template v-if="authStore.isAuthenticated">
            <!-- Notifications -->
            <NotificationBell />

            <!-- Messages -->
            <NuxtLink
              to="/messages"
              class="relative p-2 rounded-full text-gray-500 hover:text-gray-700 hover:bg-gray-100"
            >
              <Icon name="lucide:message-circle" class="w-5 h-5" />
              <span
                v-if="unreadMessages > 0"
                class="absolute top-0 right-0 block h-4 w-4 rounded-full bg-secondary-500 text-white text-xs flex items-center justify-center transform -translate-y-1/4 translate-x-1/4"
              >
                {{ unreadMessages > 9 ? '9+' : unreadMessages }}
              </span>
            </NuxtLink>

            <!-- User dropdown -->
            <UserDropdown />
          </template>
          <template v-else>
            <NuxtLink
              to="/auth/login"
              class="hidden sm:inline-flex btn-outline"
            >
              Sign in
            </NuxtLink>
            <NuxtLink
              to="/auth/register"
              class="btn-primary"
            >
              Sign up
            </NuxtLink>
          </template>
        </div>
      </div>
    </div>

    <!-- Mobile navigation menu -->
    <div
      v-if="mobileMenuOpen"
      class="md:hidden"
    >
      <div class="px-2 pt-2 pb-3 space-y-1">
        <NuxtLink
          v-for="item in navItems"
          :key="item.name"
          :to="item.href"
          :class="[
            'block px-3 py-2 rounded-md text-base font-medium',
            $route.path === item.href
              ? 'text-primary-700 bg-primary-50'
              : 'text-gray-700 hover:text-primary-600 hover:bg-gray-50'
          ]"
          @click="mobileMenuOpen = false"
        >
          {{ item.name }}
        </NuxtLink>
      </div>
    </div>
  </header>
</template>

<script setup>
import { ref } from 'vue';
import { useAuthStore } from '~/stores/auth';

const authStore = useAuthStore();
const mobileMenuOpen = ref(false);
const unreadMessages = ref(3); // Mock data - would come from an API or store

const navItems = [
  { name: 'Home', href: '/' },
  { name: 'Explore', href: '/explore' },
  { name: 'Creators', href: '/creators' },
];

function openSearch() {
  // Implementation for search functionality
  console.log('Search opened');
}
</script>