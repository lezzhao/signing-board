<script setup lang="ts">
// import * as ocr from '@paddlejs-models/ocr'
import { createDrauu } from 'drauu'
import { onMounted, ref } from 'vue'

const tempTrace = ref<DOMPoint[]>([])
const traces = ref<any[]>([])

onMounted(() => {
  const drauu = createDrauu({
    el: '#svg',
    brush: {
      mode: 'stylus', // 'line', 'rectangle', 'ellipse'
      color: 'black',
      size: 5,
    },
  })
  drauu.on('changed', () => {
    tempTrace.value.push({ x: drauu.svgPoint?.x, y: drauu.svgPoint?.y } as DOMPoint)
  })
  drauu.on('committed', () => {
    if (tempTrace.value.length > 1) {
      const traceX = tempTrace.value.map(point => point.x)
      const traceY = tempTrace.value.map(point => point.y)
      traces.value.push([traceX, traceY, []])
      console.log(traces.value)
    }
    tempTrace.value = []
  })
})
const svgEl = ref<SVGAElement>()
async function clickHandler() {
  const data = JSON.stringify({ options: 'enable_pre_space', requests: [{ writing_guide: { writing_area_width: svgEl.value?.clientWidth, writing_area_height: svgEl.value?.clientHeight }, ink: traces.value, language: 'zh_TW' }] })

  fetch('https://www.google.com.tw/inputtools/request?ime=handwriting&app=mobilesearch&cs=1&oe=UTF-8', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: data,
  }).then((res) => {
    console.log(res)
  })
}
</script>

<template>
  <button style="position: fixed; top: 20px; right: 20px;" @click="clickHandler">
    识别
  </button>
  <svg
    id="svg" ref="svgEl" style="width: 100vw; height: 100vh"
  />
</template>
