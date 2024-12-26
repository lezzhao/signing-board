<script setup lang="ts">
// import * as ocr from '@paddlejs-models/ocr'
import { createDrauu } from 'drauu'
import { onMounted, ref } from 'vue'

const tempTrace = ref<DOMPoint[]>([])
const traces = ref<any[]>([])
const content = ref('')

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
    }
    tempTrace.value = []
  })
})
const svgEl = ref<SVGAElement>()
async function clickHandler() {
  const data = JSON.stringify({ options: 'enable_pre_space', requests: [{ writing_guide: { writing_area_width: svgEl.value?.clientWidth, writing_area_height: svgEl.value?.clientHeight }, ink: traces.value, language: 'zh_TW' }] })
  const res: any[] = await fetch('https://www.google.com.tw/inputtools/request?ime=handwriting&app=mobilesearch&cs=1&oe=UTF-8', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: data,
  }).then(res => res.json())
  console.log(res.flat(Infinity))
  content.value = res.flat(6)[2]
}
</script>

<template>
  <div style="position: fixed; top: 20px; right: 20px;">
    <button @click="clickHandler">
      识别
    </button>
    <span style="margin-left: 16px;">结果： {{ content }}</span>
  </div>
  <svg
    id="svg" ref="svgEl" style="width: 100vw; height: 100vh"
  />
</template>
