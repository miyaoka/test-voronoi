<script setup lang="ts">
import { Delaunay, Voronoi } from "d3-delaunay";
import { UseDraggable } from "@vueuse/components";
import { computed, onMounted, ref } from "vue";

const vr = ref("");
const delaunay = ref(Delaunay.from([]));
const areaList = ref(
  Array.from(new Array(70), (_, i) => {
    return {
      point: [Math.random() * 960, Math.random() * 500],
      color: Math.floor(i % 6) * 60,
    };
  })
);
const draggingIdx = ref(-1);
// let dragStartPoint = [0, 0];

const voronoi = computed(() => delaunay.value.voronoi([0, 0, 960, 500]));

const onPointerMove = (e: PointerEvent) => {
  const { clientX, clientY } = e;
  // const [px, py] = points.value[draggingIdx.value];
  // const [sx, sy] = dragStartPoint;
  areaList.value[draggingIdx.value].point = [clientX, clientY];
  render();
};
const onPointerUp = (e) => {
  document.removeEventListener("pointermove", onPointerMove);
  document.removeEventListener("pointerup", onPointerUp);
};
const onPointerDown = (i: number) => {
  draggingIdx.value = i;
  // dragStartPoint = areaList.value[i].point;
  document.addEventListener("pointermove", onPointerMove);
  document.addEventListener("pointerup", onPointerUp);
};

const dr = ref("");
const render = () => {
  delaunay.value = Delaunay.from(areaList.value.map((item) => item.point));

  vr.value = voronoi.value.render();
  dr.value = delaunay.value.render();
  voronoi.value.renderCell;
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
      <path
        v-for="(area, i) in areaList"
        :key="i"
        :d="voronoi.renderCell(i)"
        :fill="`hsl(${areaList[i].color}, 50%, 80%)`"
        class="fill"
      />
      <path :d="vr" stroke="#000" />
      <path :d="dr" stroke="#ff0" fill="none" />
    </svg>
    <div
      class="draggable"
      v-for="(area, i) in areaList"
      :key="i"
      :style="{ left: `${area.point[0]}px`, top: `${area.point[1]}px` }"
      @pointerdown="onPointerDown(i)"
    >
      <div class="btn">
        {{ i }}{{ `(${[...voronoi.neighbors(i)].length})` }}
      </div>
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
  /* pointer-events: none; */
}
.btn {
  width: 4px;
  height: 4px;
  background-color: #f00;
  border-radius: 50%;
  /* border: 1px solid #ccc; */
  cursor: pointer;
  user-select: none;
}
.fill:hover {
  opacity: 0.5;
}
</style>
