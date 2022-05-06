<script setup lang="ts">
import { Delaunay, Voronoi } from "d3-delaunay";
import { UseDraggable } from "@vueuse/components";
import { onMounted, ref } from "vue";

const points = ref(
  Array.from(new Array(100), (_, i) => {
    return [Math.random() * 960, Math.random() * 500];
  })
);
const draggingIdx = ref(-1);
let dragStartPoint = [0, 0];

const onPointerMove = (e: PointerEvent) => {
  const { clientX, clientY } = e;
  // const [px, py] = points.value[draggingIdx.value];
  // const [sx, sy] = dragStartPoint;
  points.value[draggingIdx.value] = [clientX, clientY];
  render();
};
const onPointerUp = (e) => {
  console.log("up", e);

  document.removeEventListener("pointermove", onPointerMove);
  document.removeEventListener("pointerup", onPointerUp);
};
const onPointerDown = (i: number) => {
  draggingIdx.value = i;
  dragStartPoint = points.value[i];
  document.addEventListener("pointermove", onPointerMove);
  document.addEventListener("pointerup", onPointerUp);
};

const vr = ref("");
const render = () => {
  const delaunay = Delaunay.from(points.value);
  const voronoi = delaunay.voronoi([0, 0, 960, 500]);
  vr.value = voronoi.render();
};

render();
</script>

<template>
  <div class="container">
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="960"
      height="500"
      viewBox="0, 0, 960, 500"
      ref="svgEl"
    >
      <path :d="vr" stroke="#000" />
    </svg>
    <div
      class="draggable"
      v-for="(point, i) in points"
      :key="i"
      :style="{ left: `${point[0]}px`, top: `${point[1]}px` }"
      @pointerdown="onPointerDown(i)"
    >
      <div class="btn"></div>
    </div>
  </div>
</template>

<style>
.container {
  position: fixed;
  left: 0;
  top: 0;
}
.draggable {
  position: fixed;
}
.btn {
  width: 8px;
  height: 8px;
  background-color: #f00;
  border-radius: 50%;
  /* border: 1px solid #ccc; */
  cursor: pointer;
}
</style>
