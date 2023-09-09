<script setup lang="ts">
import { Ref, computed, ref } from "vue"

const files: Ref<File[]> = ref([])
const fileIndex = ref(0)

const emit = defineEmits<{
  (e: "loaded"): void
  (e: "completed"): void
}>()

const imageUrl = computed(() => {
  const file = files.value[fileIndex.value]
  if (file !== undefined) {
    return URL.createObjectURL(file)
  }

  return ""
})

function handleDragStart(event: DragEvent) {
  if (event.dataTransfer === null) {
    return
  }

  if (imageUrl.value === "") {
    return
  }

  dragging.value = true

  event.dataTransfer.setData(
    "application/url",
    JSON.stringify({
      filename: files.value[fileIndex.value].name,
      url: imageUrl.value,
    }),
  )
  event.dataTransfer.effectAllowed = "move"
  if (dragImageElement.value !== null && imageElement.value !== null) {
    dragImageElement.value.src = imageElement.value.src
    dragImageElement.value.width = imageElement.value.width
    dragImageElement.value.height = imageElement.value.height
    event.dataTransfer.setDragImage(
      dragImageElement.value,
      event.offsetX,
      event.offsetY,
    )
  }
}

function handleDragEnd() {
  dragging.value = false
}

function handleDragOver(event: DragEvent) {
  event.preventDefault()
}

function handleDrop(event: DragEvent) {
  event.preventDefault()

  const src = event.dataTransfer?.getData("application/url")
  if (src !== undefined && src !== "") {
    return
  }

  const transferFiles = event.dataTransfer?.files
  if (transferFiles !== undefined) {
    files.value = Array.from(transferFiles)
    fileIndex.value = 0
    emit("loaded")
  }
}

function next() {
  fileIndex.value += 1

  if (fileIndex.value >= files.value.length) {
    emit("completed")
  }
}

defineExpose({ next })

const dragImageElement: Ref<HTMLImageElement | null> = ref(null)
const imageElement: Ref<HTMLImageElement | null> = ref(null)

const dragging = ref(false)
</script>

<template>
  <div
    @dragover="handleDragOver"
    @drop="handleDrop"
    class="w-1/3 border-r border-l p-4 flex items-center justify-center"
  >
    <div
      class="flex flex-col gap-2 text-sm"
      v-if="fileIndex === 0 && files.length === 0"
    >
      <p>
        Ziehe hier die Bilder hin (zusammen, nicht nacheinander), die zugeordnet
        werden sollen.
      </p>
      <p>
        Die Dateinamen müssen mit "links" oder "rechts" beginnen, damit sie
        zugeordnet werden können.
      </p>
      <p>Beispiele:</p>
      <ul class="list-disc list-inside">
        <li>links_hilfe_1.png</li>
        <li>rechts_streit_2.png</li>
      </ul>
    </div>
    <div
      v-else-if="fileIndex >= files.length"
      class="flex flex-col items-center justify-center text-center gap-12"
    >
      <p class="font-bold text-4xl">SUPER!</p>
      <p class="font-bold">Du hast alle Bilder richtig zugeordnet.</p>
    </div>
    <div v-else class="flex flex-col">
      <img
        ref="dragImageElement"
        class="absolute left-[-10000px] top-[-10000px]"
        :class="dragging ? 'opacity-100' : 'opacity-0'"
      />
      <img
        v-if="imageUrl !== ''"
        :class="dragging ? 'opacity-0' : 'opacity-100'"
        ref="imageElement"
        @dragstart="handleDragStart"
        @dragend="handleDragEnd"
        draggable="true"
        :src="imageUrl"
      />
    </div>
  </div>
</template>
