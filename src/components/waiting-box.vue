<template>
  <section>
    <p v-if="maximumWaitingSeconds">
      Connect to new window... {{ `(${maximumWaitingSeconds})` }}
    </p>
    <p v-else>Waiting response...</p>
  </section>
</template>

<script setup lang="ts">
interface WaitingBoxProps {
  maximumWaitingSeconds?: number;
}

defineProps<WaitingBoxProps>();
</script>

<style scoped></style>
