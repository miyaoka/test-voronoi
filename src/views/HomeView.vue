<script setup lang="ts">
import { Delaunay, Voronoi } from "d3-delaunay";
import { UseDraggable } from "@vueuse/components";
import { computed, onMounted, ref } from "vue";

const vr = ref("");
const delaunay = ref(Delaunay.from([]));
const areaList = ref(
  Array.from(new Array(20), (_, i) => {
    return {
      id: i,
      point: [Math.random() * 960, Math.random() * 500],
      color: Math.floor(i % 6) * 60,
      rank: Math.random() * Math.random() * 10,
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
  if (isRemoveMode.value) {
    areaList.value.splice(i, 1);
    isRemoveMode.value = false;
    render();
    return;
  }
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

const isRemoveMode = ref(false);

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
        :key="area.id"
        :d="voronoi.renderCell(i)"
        :fill="`hsl(${areaList[i].color}, 50%, 80%)`"
        class="fill"
      />
      <path :d="vr" stroke="#fff" />
      <path :d="dr" stroke="#000" fill="none" style="opacity: 0.1" />
    </svg>
    <div
      class="draggable"
      v-for="(area, i) in areaList"
      :key="area.id"
      :style="{ left: `${area.point[0]}px`, top: `${area.point[1]}px` }"
      @pointerdown="onPointerDown(i)"
    >
      <div
        class="btn"
        :style="{
          width: `${Math.ceil(area.rank) * 2}px`,
          height: `${Math.ceil(area.rank) * 2}px`,
        }"
      ></div>
      <div class="name">{{ i }}{{ `(${[...voronoi.neighbors(i)]})` }}</div>
    </div>
  </div>

  <div>
    <button>add</button>
    <button @click="isRemoveMode = true">remove</button>
  </div>
</template>

<style>
.container {
}
.draggable {
  position: fixed;
  /* pointer-events: none; */
}
.btn {
  background-color: #f00;
  border-radius: 50%;
  /* border: 1px solid #ccc; */
  cursor: pointer;
  filter: blur(1px);
  transform: translate(-50%, -50%);
}
.name {
  user-select: none;
  position: absolute;
  top: 0;
}
.fill {
  filter: blur(4px);
}
.fill:hover {
  opacity: 0.5;
}
</style>
