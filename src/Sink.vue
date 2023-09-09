<script setup lang="ts">
import { X } from "lucide-vue-next"
import { Check } from "lucide-vue-next"
import { CSSProperties, Ref, ref } from "vue"
interface Props {
  isImageCorrect: (filename: string) => boolean
  placeholder: string
}

const props = defineProps<Props>()

const emit = defineEmits<{
  (e: "imageAccepted"): void
}>()

function handleDragOver(event: DragEvent) {
  style.value.boxShadow = "inset 0px 0px 0px 4px #aaa"
  event.preventDefault()
}

function handleDragLeave() {
  delete style.value.boxShadow
}

function handleDrop(event: DragEvent) {
  delete style.value.boxShadow
  event.preventDefault()

  const data = event.dataTransfer?.getData("application/url")

  if (data !== undefined && data !== "") {
    const { filename } = JSON.parse(data)
    if (props.isImageCorrect(filename)) {
      flashCorrect()
      emit("imageAccepted")
    } else {
      flashWrong()
    }
    return
  }

  const file = event.dataTransfer?.files.item(0)
  if (file) {
    if (style.value.backgroundImage !== undefined) {
      URL.revokeObjectURL(style.value.backgroundImage)
    }

    style.value.backgroundImage = `url(${URL.createObjectURL(file)})`
  }
}

async function flashCorrect() {
  if (flashCorrectElement.value?.classList.contains("flash1")) {
    flashCorrectElement.value?.classList.remove("flash1")
    flashCorrectElement.value?.classList.add("flash2")
  } else {
    flashCorrectElement.value?.classList.remove("flash2")
    flashCorrectElement.value?.classList.add("flash1")
  }
}

async function flashWrong() {
  if (flashWrongElement.value?.classList.contains("flash1")) {
    flashWrongElement.value?.classList.remove("flash1")
    flashWrongElement.value?.classList.add("flash2")
  } else {
    flashWrongElement.value?.classList.remove("flash2")
    flashWrongElement.value?.classList.add("flash1")
  }
}

const flashCorrectElement: Ref<HTMLDivElement | null> = ref(null)
const flashWrongElement: Ref<HTMLDivElement | null> = ref(null)
const style: Ref<CSSProperties> = ref({})
</script>

<template>
  <div
    @drop="handleDrop"
    @dragleave="handleDragLeave"
    @dragover="handleDragOver"
    class="relative bg-contain bg-no-repeat bg-center"
    :style="style"
  >
    <div
      v-if="style.backgroundImage === undefined"
      class="flex w-full h-full text-sm flex-col items-center justify-center"
    >
      <p class="text-center">{{ placeholder }}</p>
    </div>
    <div
      ref="flashWrongElement"
      class="opacity-0 absolute pointer-events-none flex items-center justify-center w-full h-full left-0 top-0 bg-red-500"
    >
      <X :size="200" />
    </div>
    <div
      ref="flashCorrectElement"
      class="opacity-0 absolute pointer-events-none flex items-center justify-center w-full h-full left-0 top-0 bg-green-500"
    >
      <Check :size="200" />
    </div>
  </div>
</template>

<style scoped>
.flash1 {
  animation: flash1 0.7s linear;
}

@keyframes flash1 {
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

.flash2 {
  animation: flash2 0.7s linear;
}

@keyframes flash2 {
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
