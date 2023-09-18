<script setup lang="ts">
import { Upload, X, Image } from "lucide-vue-next"
import { Check } from "lucide-vue-next"
import { CSSProperties, Ref, ref } from "vue"
interface Props {
  hovered: boolean
}

const props = defineProps<Props>()

const emit = defineEmits<{
  (e: "imageLoaded", url: string): void
}>()

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

function handleFileChange(event: Event) {
  if (event.target === null) {
    return
  }
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (file === undefined) {
    return
  }

  if (style.value.backgroundImage !== undefined) {
    URL.revokeObjectURL(style.value.backgroundImage)
  }

  const url = URL.createObjectURL(file)

  style.value.backgroundImage = `url(${url})`

  emit("imageLoaded", url)
}

const flashCorrectElement: Ref<HTMLDivElement | null> = ref(null)
const flashWrongElement: Ref<HTMLDivElement | null> = ref(null)
const style: Ref<CSSProperties> = ref({})

defineExpose({
  flashCorrect,
  flashWrong,
})
</script>

<template>
  <div
    class="relative bg-contain bg-no-repeat bg-center items-center justify-center"
    :class="hovered ? 'highlight' : ''"
    :style="style"
  >
    <input
      v-if="style.backgroundImage === undefined"
      type="file"
      class="absolute w-full h-full opacity-0 cursor-pointer"
      @change="handleFileChange"
    />
    <div
      v-if="style.backgroundImage === undefined"
      class="flex w-full h-full text-sm flex-col items-center justify-center p-8"
    >
      <div
        class="w-full gap-4 text-gray-500 border-gray-500 border-2 aspect-square items-center flex-col justify-center flex border-dashed rounded-xl"
      >
        <Image :size="100" />
        <Upload :size="40" />
        Bild hochladen
      </div>
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

.highlight {
  box-shadow: inset 0px 0px 0px 4px #aaa;
}
</style>
