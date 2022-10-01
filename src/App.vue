<script setup lang="ts">
import { ref } from "vue";

// Types
interface SendMessage {
  data: string;
  command: "open" | "data";
}

// Variables
const targetOrigin =
  import.meta.env.MODE === "development"
    ? "http://localhost:5175"
    : "https://codeliners-post-message-window-b.netlify.app";
let newWindow: Window | null = null;
let intervalId: number;
let response = ref();

// Initialize.
window.addEventListener("message", receiveMessage, false);

// Functions
function receiveMessage(event: MessageEvent) {
  const { data, origin } = event;
  if (origin !== targetOrigin) return;

  const parsedData = JSON.parse(data) as SendMessage;
  if (parsedData.command === "open") {
    console.log("[receiveMessage] Connected.", data, origin);
    // Clear interval.
    clearInterval(intervalId);
    return;
  }

  response.value = parsedData.data;
}

function sendMessage({ data, command }: SendMessage) {
  if (!newWindow) return;
  newWindow.postMessage(JSON.stringify({ data, command }), targetOrigin);
}

function openPopup() {
  newWindow = window.open(targetOrigin);
  intervalId = setInterval(() => {
    sendMessage({ data: "open", command: "open" });
  }, 1000);
}
</script>

<template>
  <div>
    <button @click="openPopup">Open new window</button>
    <p v-if="response">Response {{ response }}</p>
  </div>
</template>

<style scoped></style>
