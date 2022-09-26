<script setup lang="ts">
const el = ref<HTMLCanvasElement>()
const ctx = $computed(() => el.value!.getContext('2d')!)
const WIDTH = 500
const HEIGHT = 500
let pendingTasks: Function[] = []

interface Point {
  x: number
  y: number
}
interface Branch {
  start: Point
  length: number
  theta: number
}

function init() {
  ctx.strokeStyle = 'rgb(156, 163, 175, 0.5)'
  step({
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 5,
    theta: Math.PI / 2,
  })
}
function step(b: Branch, depth = 0) {
  drawBranch(b)
  const startP = getEndPoint(b)
  if (depth < 5 || Math.random() < 0.5) {
    pendingTasks.push(() => {
      step({
        start: startP,
        length: b.length + Math.random(),
        theta: b.theta - 0.3 * Math.random(),
      }, depth + 1)
    })
  }
  if (depth < 5 || Math.random() < 0.5) {
    pendingTasks.push(() => {
      step({
        start: startP,
        length: b.length,
        theta: b.theta + 0.3 * Math.random(),
      }, depth + 1)
    })
  }
}
let framesCount = 0
function startAnimate() {
  requestAnimationFrame(() => {
    framesCount += 1
    if (framesCount % 30 === 0)
      frame()
    startAnimate()
  })
}
function frame() {
  const tasks: Function[] = []

  pendingTasks = pendingTasks.filter((i) => {
    if (Math.random() > 0.4) {
      tasks.push(i)
      return false
    }
    return true
  })
  tasks.forEach(fn => fn())
}

startAnimate()
function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}

function getEndPoint(b: Branch): Point {
  const { start, length, theta } = b
  return {
    x: start.x - length * Math.cos(theta),
    y: start.y - length * Math.sin(theta),
  }
}

function lineTo(sp: Point, ep: Point) {
  ctx.beginPath()
  ctx.moveTo(sp.x, sp.y)
  ctx.lineTo(ep.x, ep.y)
  ctx.stroke()
}
onMounted(() => {
  init()
})
</script>

<template>
  <main font-sans p="x-4 y-10" text="center gray-700 dark:gray-200">
    <div flex="~" justify-center>
      <canvas
        ref="el"
        :width="WIDTH"
        :height="HEIGHT"
        border
      />
    </div>
    <Footer />
  </main>
</template>
