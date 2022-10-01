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
    ? "http://localhost:5556"
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
  const widthValue = 400;
  const heightValue = widthValue;
  const leftValue = window.screen.availWidth / 2 - widthValue / 2;
  const topValue = window.screen.availHeight / 2 - heightValue / 2;

  const windowFeatures = `popup,toolbar,scrollbars=yes,top=${topValue},left=${leftValue},width=${widthValue},height=${heightValue}`;

  newWindow = window.open(targetOrigin, "_blank", windowFeatures);

  intervalId = window.setInterval(() => {
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
