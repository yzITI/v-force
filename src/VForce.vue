<script setup>
import { onMounted } from 'vue'
const { width, height } = defineProps(['width', 'height'])

const N = 20, m = 1, k = 0.1, c = 200, β = 1
let nodes = $ref([]), links = $ref([]), id2node = {}
let onHand = $ref(''), svg = $ref(), pt = {}

onMounted(() => { pt = svg.createSVGPoint() })

for (let i = 1; i <= N; i++) nodes.push({
  id: Math.random().toString(36).substr(2),
  x: 100*Math.random(), y: 100*Math.random(),
  vx: 0, vy: 0, fx: 0, fy: 0
})
for (const n of nodes) id2node[n.id] = n
for (let i = 1; i <= 1.5*N; i++) {
  const randpos = () => Math.floor(Math.random()*N)
  links.push({
    id: Math.random().toString(36).substr(2),
    from: nodes[randpos()].id,
    to: nodes[randpos()].id
  })
}
const dist = (n, m) => Math.sqrt((m.x-n.x)*(m.x-n.x) + (m.y-n.y)*(m.y-n.y))

async function loop () {
  let maxv = 0, Δ = 1
  while (1) {
    maxv = 0
    for (const n of nodes) n.fx = n.fy = 0
    for (let i = 0; i < nodes.length; i++) {
      for (let j = i+1; j < nodes.length; j++) {
        const n = nodes[i], m = nodes[j]
        const d = dist(m, n)
        const f = Math.min(c/(d*d), c*1000)
        m.fx += f * (m.x-n.x)/d
        m.fy += f * (m.y-n.y)/d
        n.fx += f * (n.x-m.x)/d
        n.fy += f * (n.y-m.y)/d
      }
    }
    for (const l of links) {
      const n = id2node[l.from], m = id2node[l.to]
      n.fx += k * (m.x - n.x)
      n.fy += k * (m.y - n.y)
      m.fx += k * (n.x - m.x)
      m.fy += k * (n.y - m.y)
    }
    for (const n of nodes) {
      n.fx -= n.vx * β
      n.fy -= n.vy * β
      n.vx += n.fx/m
      n.vy += n.fy/m
      maxv = Math.max(maxv, n.vx)
      maxv = Math.max(maxv, n.vy)
      if (n.id == onHand) continue
      n.x += n.vx
      n.y += n.vy
    }
    Δ = Math.min(Math.max(1/Math.sqrt(maxv), 1), 20)
    await new Promise(r => setTimeout(r, Δ*20))
  }
}
loop()

function drag (e) {
  if (!onHand) return
  const n = id2node[onHand]
  pt.x = e.clientX
  pt.y = e.clientY
  const p = pt.matrixTransform(svg.getScreenCTM().inverse())
  n.x = p.x
  n.y = p.y
}
</script>

<template>
  <div class="v-force">
    <svg ref="svg" viewBox="0 0 100 100" :width="width" :height="height" @mouseup="onHand = ''" @mousemove="drag">
      <line v-for="l in links" :x1="id2node[l.from].x" :y1="id2node[l.from].y" :x2="id2node[l.to].x" :y2="id2node[l.to].y" stroke="black" stroke-width="0.5"/>
      <circle v-for="n in nodes" :cx="n.x" :cy="n.y" r="2" fill="red" @mousedown="onHand = n.id"/>
    </svg>
  </div>
</template>
