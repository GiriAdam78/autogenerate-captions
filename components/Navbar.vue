<script setup lang="ts">
const searchTerm = ref([
  {
    id: "menu",
    items: [
      {
        label: "Home",
        icon: "i-lucide-home",
      },
      {
        label: "Github",
        icon: "i-lucide-github",
      },
    ],
  },
]);
</script>
<template>
  <nav
    class="sticky top-0 w-full shadow-sm h-[60px] flex items-center bg-white/30 backdrop-blur-sm justify-between @container"
  >
    <h1 class="font-sans ml-3.5">Auto Generate Captions</h1>

    <UModal>
      <UButton
        color="neutral"
        variant="ghost"
        icon="i-lucide-search"
        class="cursor-pointer ml-3"
      ></UButton>
      <template #content>
        <UCommandPalette
          icon="i-lucide-search"
          :groups="searchTerm"
          class="flex-1"
        />
      </template>
    </UModal>
  </nav>
</template>
