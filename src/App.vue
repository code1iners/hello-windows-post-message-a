<script setup lang="ts">
import { ref } from "vue";
import { usePopup } from "@ce1pers/use-window";
import type { SendMessage } from "@ce1pers/use-window/dist/src/usePopup/types";
import WaitingBox from "@/components/waiting-box.vue";
import ResponseBox from "@/components/response-box.vue";

// Hooks.
const { open, sendMessageToTargetOrigin } = usePopup({
  onMessageCallback,
});

// Variables
const EVERY_SECOND = 1000;
const MAXIMUM_TIMER_SECONDS = EVERY_SECOND * 10;

const targetOrigin =
  import.meta.env.MODE === "development"
    ? "http://localhost:5556"
    : "https://codeliners-post-message-window-b.netlify.app";
let intervalId: number;
const response = ref();
const isWaitingResponse = ref(false);
const maximumWaitingSeconds = ref<undefined | number>(undefined);

// Functions
function clearTimer() {
  clearInterval(intervalId);
  maximumWaitingSeconds.value = undefined;
}

function setIsWaitingResponse(status: boolean) {
  isWaitingResponse.value = status;
}

function onMessageCallback(event: MessageEvent) {
  // Destructuring.
  const { data, origin } = event;
  if (origin !== targetOrigin) return;

  const parsedData = JSON.parse(data) as SendMessage;

  switch (parsedData.type) {
    case "open":
      clearTimer();
      // Send message for hand shake.
      sendMessageToTargetOrigin({ type: "ready", data: "Ready" });
      break;

    case "close":
      setIsWaitingResponse(false);
      break;

    case "data":
      response.value = parsedData.data;
      setIsWaitingResponse(false);
      break;
  }
}

function openPopupCallback() {
  // Set variables.
  setIsWaitingResponse(true);
  maximumWaitingSeconds.value = 10;

  // Attempts to reconnect with opened window every second.
  intervalId = window.setInterval(() => {
    sendMessageToTargetOrigin({ data: "open", type: "open" });

    if (maximumWaitingSeconds.value) {
      maximumWaitingSeconds.value -= 1;
    }
  }, EVERY_SECOND);

  // Control retry maximum count.
  window.setTimeout(() => {
    if (isWaitingResponse.value) {
      alert("Failed to connect to new window.");
      clearTimer();
      setIsWaitingResponse(false);
    }
  }, MAXIMUM_TIMER_SECONDS);
}
</script>

<template>
  <article>
    <WaitingBox
      v-if="isWaitingResponse"
      :maximum-waiting-seconds="maximumWaitingSeconds"
    />
    <ResponseBox
      v-else
      :response="response"
      :open-popup="
        () =>
          open({
            targetOrigin,
            windowTarget: '_blank',
            callback: openPopupCallback,
            width: 400,
            height: 400,
          })
      "
    />
  </article>
</template>

<style scoped></style>
