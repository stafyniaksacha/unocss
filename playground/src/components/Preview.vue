<script setup lang="ts">
import { isDark } from '../logics/dark'
import { useResize } from '../useResize'
import { init, output, transformedHTML } from '../logics/uno'
import { options } from '../logics/url'

const iframe = ref<HTMLIFrameElement>()

const iframeData = reactive({
  source: 'unocss-playground',
  css: computed(() => output.value?.css || ''),
  html: transformedHTML,
  dark: isDark,
})

async function send() {
  iframe.value?.contentWindow?.postMessage(JSON.stringify(iframeData), location.origin)
}

watch([iframeData, iframe], send, { deep: true })

const canvasRef = ref()
const frameRef = ref()
const scale = ref(1)
const { width, height, isResizing, style, onResizeEnd } = useResize(frameRef, {
  borderRadius: 8,
  xMultiplier: computed(() => 2 / scale.value),
  yMultiplier: computed(() => 1 / scale.value),
  minWidth: 270,
  minHeight: 300,
  mode: 'manual',
  edgeWidth: computed(() => options.value.responsive ? [0, 16] : [0, 0]),
})

const size = useElementSize(canvasRef)
const { width: frameWidth, height: frameHeight, update } = useElementBounding(frameRef)

watch([size.width, size.height, width, height], () => {
  const scaleSize = Math.min(size.width.value / width.value, size.height.value / height.value)
  scale.value = scaleSize > 1 ? 1 : scaleSize

  setTimeout(() => update(), 0)
})
const isResponsive = computed(() => options.value.responsive)
onMounted(() => {
  setTimeout(() => {
    if (isResponsive.value && options.value.width && options.value.height) {
      width.value = options.value.width
      height.value = options.value.height
    }
    update()
  }, 0)
})
let resized = false
onResizeEnd(() => {
  if (isResponsive.value) {
    resized = true
    options.value.width = width.value
    options.value.height = height.value
  }
})
watch(isResponsive, (responsive) => {
  if (!responsive) {
    delete options.value.width
    delete options.value.height
  }
  else {
    if (!resized) {
      width.value = 375
      height.value = 706
    }
    options.value.width = width.value
    options.value.height = height.value
  }
})
</script>

<template>
  <div
    ref="canvasRef"
    class="h-screen overflow-hidden flex justify-center w-full bg-light-900 dark:bg-dark-900 relative"
    :class="{ 'p-4': options.responsive, 'pointer-events-none': isResizing }"
  >
    <div v-if="options.responsive" class="absolute flex items-start" :style="`width:${frameWidth}px;height:${frameHeight}px`">
      <div class="absolute -mt-4 h-4 w-full flex items-center justify-center bg-light-700 dark:bg-dark-800">
        <div class="text-xs font-medium text-gray-600 dark:text-gray-400">
          {{ width.toFixed(0) }}x{{ height.toFixed(0) }} ({{ (100 * scale).toFixed(0) }}%)
        </div>
      </div>
      <div class="absolute -ml-4 h-full w-4 flex items-center justify-center bg-light-700 dark:bg-dark-800">
        <span w-4 h-4 i-la-grip-lines-vertical text-gray-400 />
      </div>
      <div class="absolute right-0 -mr-4 h-full w-4 flex items-center justify-center bg-light-700 dark:bg-dark-800">
        <span w-4 h-4 i-la-grip-lines-vertical text-gray-400 />
      </div>
      <div class="absolute bottom-0 -mb-4 h-4 w-full flex items-center justify-center bg-light-700 dark:bg-dark-800">
        <span w-4 h-4 i-la-grip-lines text-gray-400 />
      </div>
      <div class="absolute right-0 bottom-0 -mr-4 -mb-4 h-4 w-4 flex items-center justify-center bg-light-700 dark:bg-dark-800">
        <span w-4 h-4 i-la-grip-lines text-gray-400 class="-rotate-45 -mt-1 -ml-1" />
      </div>
      <div class="absolute left-0 bottom-0 -ml-4 -mb-4 h-4 w-4 flex items-center justify-center bg-light-700 dark:bg-dark-800">
        <span w-4 h-4 i-la-grip-lines text-gray-400 class="rotate-45 -mt-1 -mr-1" />
      </div>
      <div class="absolute top-0 left-0 -ml-4 -mt-4 h-4 w-4 flex items-center justify-center bg-light-700 dark:bg-dark-800">
        <span w-4 h-4 i-la-grip-lines text-gray-400 class="rotate-135 -mb-1 -mr-1" />
      </div>
      <div class="absolute top-0 right-0 -mr-4 -mt-4 h-4 w-4 flex items-center justify-center bg-light-700 dark:bg-dark-800">
        <span w-4 h-4 i-la-grip-lines text-gray-400 class="rotate-45 -mb-1 -ml-1" />
      </div>
    </div>
    <div
      ref="frameRef"
      class="relative w-full h-full bg-white"
      :style="options.responsive ? `${style}transform:scale(${scale});transform-origin: top center;` : ''"
    >
      <div
        :style="options.responsive ? style : ''"
        class="overflow-auto"
        min-w-0 min-h-0 w-full h-full
      >
        <iframe
          v-show="init"
          ref="iframe"
          border-0 flex-grow min-w-0 w-full h-full min-h-0
          :class="{ 'dark': isDark, 'pointer-events-none': isResizing }"
          src="/play/__play.html"
          @load="send"
        />
      </div>
    </div>
  </div>
</template>
