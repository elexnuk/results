<template>
  
  <div>
    <Title></Title>
    <ElectionResult :result="focusResult" ></ElectionResult>
  </div>
  
  <Map :dataSource="dataSrc" mapType="real" displayType="majority" @changeFocus="changeFocus"></Map>
</template>

<script setup>
import { ref, onMounted, computed, shallowReactive, reactive } from '@vue/runtime-core';
import * as d3 from "d3";

import Map from "./components/Map.vue";
import ElectionResult from "./components/ElectionResult.vue";
import Title from "./components/Title.vue";

let WestminsterConst = shallowReactive({ "value": {} });
let Election2019 = shallowReactive({ "value": {} });

let state = reactive({focusOn: ""});

/**
 * Event Handler for changing the active constituency to display results for
 */
function changeFocus (newFocus) {
  state.focusOn = newFocus;
  console.log(focusResult.value);
}

onMounted(async () => {
  WestminsterConst.value = await d3.json("./WestminsterConst.geojson");
  Election2019.value = await d3.json("./2019.json");
});

const dataSrc = computed(() => {
  return { "boundary": WestminsterConst, "election": Election2019 };
});

const focusResult = computed(() => {
  return dataSrc.value.election.value[state.focusOn] || {};
});

</script> 

<style>

</style>
