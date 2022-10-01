<template>
  <section>
    <button @click="openPopup">Open new window</button>
    <p v-if="response">Response {{ response }}</p>
  </section>
</template>

<script setup lang="ts">
interface ResponseBoxProps {
  response?: any;
  openPopup: () => void;
}

const { response, openPopup } = defineProps<ResponseBoxProps>();
</script>

<style scoped></style>
