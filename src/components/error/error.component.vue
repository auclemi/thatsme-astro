<template>
  <div
    class="alert alert-warning text-center p-4 rounded shadow-sm mb-5"
    role="alert"
  >
    <div class="fs-1">😵‍💫</div>
    <h1 class="mt-2">Oops! Quelque chose s'est mal passé.</h1>
    <p class="mb-2 error-message">{{ errorMessage }}</p>
    <small class="text-muted">{{ instruction }}</small>
  </div>
</template>
<script setup lang="ts">
defineProps<{
  errorMessage: string;
  instruction?: string;
}>();
</script>
