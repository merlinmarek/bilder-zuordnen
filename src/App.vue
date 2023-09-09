<script setup lang="ts">
import { Ref, ref } from "vue"
import Sink from "./Sink.vue"
import Source from "./Source.vue"
import ConfettiExplosion from "vue-confetti-explosion"

function isLeftImage(filename: string): boolean {
  return filename.startsWith("links")
}

function isRightImage(filename: string): boolean {
  return filename.startsWith("rechts")
}

const sourceElement: Ref<typeof Source | null> = ref(null)

function handleImageAccepted() {
  sourceElement.value?.next()
}

const showConfetti = ref(false)
</script>

<template>
  <div class="flex w-screen h-screen overflow-hidden">
    <Sink
      :is-image-correct="isLeftImage"
      class="w-1/3"
      @image-accepted="handleImageAccepted"
      placeholder="Ziehe hier das Bild hin, das die linke Seite darstellt"
    />
    <Source
      ref="sourceElement"
      class="w-1/3"
      @completed="showConfetti = true"
      @loaded="showConfetti = false"
    />
    <Sink
      :is-image-correct="isRightImage"
      class="w-1/3"
      @image-accepted="handleImageAccepted"
      placeholder="Ziehe hier das Bild hin, das die rechte Seite darstellt"
    />
    <div
      class="flex flex-col w-full h-full absolute items-center justify-start pointer-events-none overflow-hidden pt-40"
    >
      <!-- @vue-skip -->
      <ConfettiExplosion v-if="showConfetti" :particle-count="200" />
    </div>
  </div>
</template>

<style>
.flash {
  animation: flash 0.7s linear;
}

@keyframes flash {
  0% {
    opacity: 0%;
  }
  30% {
    opacity: 100%;
  }
  70% {
    opacity: 100%;
  }
  100% {
    opacity: 0%;
  }
}
</style>
