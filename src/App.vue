<template>
  <div
    class="m-4 mb-0 p-2 text-gradient text-transparent bg-clip-text w-min pointer-events-none select-none"
  >
    <h3 class="font-mono font-bold text-3xl">jamesm2w/</h3>
    <h1 class="font-mono font-semibold text-5xl">
      Election
      <br />Results
    </h1>
  </div>
  <Map :dataSource="dataSrc" mapType="real" displayType="majority"></Map>
</template>

<script setup>
import Map from "./components/Map.vue";
import * as d3 from "d3";

import { ref, onMounted, computed, shallowReactive } from '@vue/runtime-core';

let WestminsterConst = shallowReactive({ "value": {} });
let Election2019 = shallowReactive({ "value": {} });

onMounted(async () => {
  WestminsterConst.value = await d3.json("./WestminsterConst.geojson");
  Election2019.value = await d3.json("./2019.json");
});

const dataSrc = computed(() => {
  return { "boundary": WestminsterConst, "election": Election2019 };
});

</script> 

<style>

</style>
