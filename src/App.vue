<script setup lang="ts">
import { Upload, Image } from "lucide-vue-next"
import { Ref, computed, ref } from "vue"
import Sink from "./Sink.vue"
import ConfettiExplosion from "vue-confetti-explosion"

const files: Ref<File[]> = ref([])
const fileIndex = ref(0)

const showConfetti = ref(false)

const imageUrl = computed(() => {
  const file = files.value[fileIndex.value]
  if (file !== undefined) {
    return URL.createObjectURL(file)
  }

  return ""
})

const filename = computed(() => {
  const file = files.value[fileIndex.value]
  if (file !== undefined) {
    return file.name
  }

  return ""
})

const isLeft = computed(() => filename.value.startsWith("links"))
const isRight = computed(() => filename.value.startsWith("rechts"))

function handleFileChange(event: Event) {
  if (event.target === null) {
    return
  }

  const target = event.target as HTMLInputElement

  if (target.files === null) {
    return
  }

  files.value = Array.from(target.files)
  fileIndex.value = 0
}

let mouseMoving = false

function handleTouchStart(event: TouchEvent) {
  event.preventDefault()

  mouseMoving = true

  document.addEventListener("touchmove", handleTouchMove)
  document.addEventListener("touchend", handleTouchEnd)
}

function handleTouchMove(event: TouchEvent) {
  const touch = event.touches[0]
  if (touch === undefined) {
    return
  }

  const x = touch.clientX
  const y = touch.clientY

  left.value = Math.max(
    -window.innerWidth / 2,
    Math.min(window.innerWidth / 2, x - window.innerWidth / 2),
  )
  top.value = Math.min(
    window.innerHeight / 2,
    Math.max(-window.innerHeight / 2, y - window.innerHeight / 2),
  )
}

async function handleTouchEnd(_event: TouchEvent) {
  document.removeEventListener("touchmove", handleTouchMove)
  document.removeEventListener("touchend", handleTouchEnd)

  if (hoverLeft.value) {
    if (isLeft.value) {
      fileIndex.value += 1
      leftSinkElement.value?.flashCorrect()
    } else {
      leftSinkElement.value?.flashWrong()
    }
  }

  console.log(hoverRight.value, isRight.value, filename.value)
  if (hoverRight.value) {
    if (isRight.value) {
      fileIndex.value += 1
      rightSinkElement.value?.flashCorrect()
    } else {
      rightSinkElement.value?.flashWrong()
    }
  }

  left.value = 10000000
  top.value = 10000000

  await new Promise((r) => setTimeout(r, 500))

  left.value = 0
  top.value = 0

  if (fileIndex.value >= files.value.length) {
    showConfetti.value = true
  }
}

function handleMouseDown(event: MouseEvent) {
  event.preventDefault()
  mouseMoving = true

  document.addEventListener("mousemove", handleMouseMove)
  document.addEventListener("mouseup", handleMouseUp)
}

function handleMouseMove(event: MouseEvent) {
  left.value = Math.max(
    -window.innerWidth / 2,
    Math.min(window.innerWidth / 2, event.clientX - window.innerWidth / 2),
  )
  top.value = Math.min(
    window.innerHeight / 2,
    Math.max(-window.innerHeight / 2, event.clientY - window.innerHeight / 2),
  )
}

async function handleMouseUp(_event: MouseEvent) {
  document.removeEventListener("mousemove", handleMouseMove)
  document.removeEventListener("mouseup", handleMouseUp)

  if (hoverLeft.value) {
    if (isLeft.value) {
      fileIndex.value += 1
      leftSinkElement.value?.flashCorrect()
    } else {
      leftSinkElement.value?.flashWrong()
    }
  }

  console.log(hoverRight.value, isRight.value, filename.value)
  if (hoverRight.value) {
    if (isRight.value) {
      fileIndex.value += 1
      rightSinkElement.value?.flashCorrect()
    } else {
      rightSinkElement.value?.flashWrong()
    }
  }

  left.value = 10000000
  top.value = 10000000

  await new Promise((r) => setTimeout(r, 500))

  left.value = 0
  top.value = 0

  if (fileIndex.value >= files.value.length) {
    showConfetti.value = true
  }
}

const left = ref(0)
const top = ref(0)

const hoverLeft = computed(() => left.value < 0 && mouseMoving)
const hoverRight = computed(() => left.value >= 0 && mouseMoving)

const leftSinkElement = ref<typeof Sink>()
const rightSinkElement = ref<typeof Sink>()

let leftBackgroundUrl = ""
</script>

<template>
  <div class="flex w-screen h-screen overflow-hidden">
    <div
      v-if="showConfetti"
      class="flex h-screen w-full flex-col items-center justify-center gap-8 px-8 py-8"
    >
      <p class="text-4xl font-bold">Gemeinsam sind wir stark!</p>
      <!-- @vue-skip -->
      <div
        class="absolute h-full w-full flex items-center justify-center overflow-hidden"
      >
        <ConfettiExplosion class="top-[-100px]" :particleCount="200" />
      </div>
      <div
        :style="{ backgroundImage: `url(${leftBackgroundUrl})` }"
        class="grow bg-center bg-contain w-full bg-no-repeat"
      ></div>
    </div>
    <div
      v-if="!showConfetti"
      class="absolute flex w-full h-full justify-center items-center overflow-hidden"
    >
      <Sink
        ref="leftSinkElement"
        :hovered="hoverLeft"
        class="w-1/2 h-full border-r"
        @image-loaded="(url) => (leftBackgroundUrl = url)"
      />
      <Sink
        ref="rightSinkElement"
        :hovered="hoverRight"
        class="w-1/2 h-full border-l"
      />
      <div
        v-if="imageUrl !== ''"
        class="absolute w-1/3 aspect-square flex items-center justify-center"
        @mousedown="handleMouseDown"
        @touchstart="handleTouchStart"
      >
        <img
          :style="{
            left: `${left}px`,
            top: `${top}px`,
            boxShadow: `0px 0px 12px
          black`,
          }"
          class="rounded-xl bg-white absolute border-4"
          :src="imageUrl"
        />
      </div>
      <div
        v-if="imageUrl === '' && files.length === 0"
        class="absolute w-1/4 aspect-square bg-white flex items-center justify-center shadow-xl rounded-xl"
      >
        <div
          class="absolute bg-white gap-4 text-gray-500 border-gray-500 border-2 w-full aspect-square items-center flex-col justify-center flex border-dashed rounded-xl text-sm"
        >
          <Image :size="70" />
          <Upload :size="30" />
          Bilder hochladen
        </div>
        <input
          type="file"
          multiple
          class="absolute w-full h-full opacity-0 cursor-pointer"
          @change="handleFileChange"
        />
      </div>
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
