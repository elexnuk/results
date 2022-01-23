<template>
  
  <div>
    <div class="flex justify-between items-center flex-wrap">
      <Title></Title>
      <div class="mr-4 my-2 bg-slate-400 p-4 rounded-md shadow-md flex flex-wrap items-center justify-between space-x-2 w-3/5 gap-2 flex-1"> 
        
        <SelectionList :election="dataSrc.election"></SelectionList>

        <div class="basis-full">
          <label for="selectPath">Search: </label>
          <input name="selectPath" class="rounded-sm p-1" v-model="search.focusOn" placeholder="Start Typing" list="selectionList"> 
        </div>
        
        <div class="flex-1">
          <label for="selectElectionCode">Election: </label>
          <select name="selectElectionCode" class="cursor-pointer w-full rounded-sm p-1" v-model="selectedElectionData">
            <option v-for="election of Object.keys(electionData)" :key="election">{{ election }}</option>
          </select> 
        </div>

        <div class="flex-1">
          <label for="selectMapShading">Display: </label>
          <select name="selectMapShading" class="cursor-pointer w-full rounded-sm p-1" v-model="selectedMapShading" value="winner">
            <option v-for="name of Object.keys(mapShadings)" :key="mapShadings[name]" :value="mapShadings[name]">{{name}}</option>
          </select> 
        </div>

        <div class="flex-1">
          <label for="selectMapFilter">Filter: </label>
          <select name="selectMapFilter" class="cursor-pointer w-full rounded-sm p-1" v-model="selectedMapFilter" value="all">
            <option v-for="name of Object.keys(mapFilters)" :key="mapFilters[name]" :value="mapFilters[name]">{{name}}</option>
          </select> 
        </div>
      </div>
    </div>
    <ElectionResult :result="focusResult" ></ElectionResult>
  </div>
  
  <Map
    :boundary="dataSrc.boundary" 
    :election="dataSrc.election" 
    tagProperty="PCON20CD" 
    :displayType="displayType"
    :focusPath="search"
    @changeFocus="changeFocus"
  ></Map>

  
  <div class="w-full bg-slate-600 rounded-lg shadow-lg p-4 text-gray-100" style="grid-column: 1/-1;">
    Wesbite licensed under the <a class="underline hover:no-underline text-red-500 hover:text-red-200 transition-all ease-in" href="https://raw.githubusercontent.com/jamesm2w/election-results/main/LICENSE">GNU AGPLv3</a>.
    (<a class="underline hover:no-underline text-blue-500 hover:text-blue-200 transition-all ease-in" href="https://github.com/jamesm2w/election-results">Github Source</a>).
    <br>
    Election Result information licensed under the <a class="underline hover:no-underline text-red-500 hover:text-red-200 transition-all ease-in" href="https://www.parliament.uk/site-information/copyright-parliament/open-parliament-licence/">Open Parliament Licence v3.0</a> 
    (<a class="underline hover:no-underline text-blue-500 hover:text-blue-200 transition-all ease-in" href="https://electionresults.parliament.uk/">Results Source</a>).
    <br>
    UK Constituency Boundaries Source: Office for National Statistics licensed under the  <a class="underline hover:no-underline text-red-500 hover:text-red-200 transition-all ease-in" href="https://www.ons.gov.uk/methodology/geography/licences">Open Government Licence v.3.0</a> 
    (<a class="underline hover:no-underline text-blue-500 hover:text-blue-200 transition-all ease-in" href="https://geoportal.statistics.gov.uk/datasets/westminster-parliamentary-constituencies-december-2020-uk-bgc/explore?location=55.340085%2C-3.316534%2C6.04">Boundary Source</a>).
  </div>
</template>

<script setup>
import { ref, onMounted, computed, shallowReactive, reactive } from '@vue/runtime-core';
import * as d3 from "d3";

/* Import compoentns */
import Map from "./components/Map.vue";
import ElectionResult from "./components/ElectionResult.vue";
import Title from "./components/Title.vue";
import SelectionList from './components/SelectionList.vue';

/* Reactive structures for the JSON data */
let WestminsterConst = shallowReactive({ "value": {} }); // 2010-202X Boundary Data
let Election2019 = shallowReactive({ "value": {} }); // GE2019 Election Results
let ByElection2019 = shallowReactive({ "value": {} }); // By-Elections 2019-202X Results

let Election2017 = shallowReactive({ "value": {} });
let Election2015 = shallowReactive({ "value": {} });
let ByElection2017 = shallowReactive({ "value": {} });
let ByElection2015 = shallowReactive({ "value": {} });

let state = reactive({focusOn: ""});
let search = reactive({focusOn: ""});
let selectedElectionData = ref("GE2019");
let selectedMapShading = ref("winner");
let selectedMapFilter = ref("all");

let displayType = reactive({
  "fill": selectedMapShading,
  "filter": selectedMapFilter
});

const mapShadings = {
  "Default": "winner",
  "Runner-Up": "second",
  "Majority Shaded": "majority",
  "Turnout Shaded": "turnout"
}

const mapFilters = {
  "All": "all",
  "Won on Majority": "majority",
  "Won on Plurality": "plurality"
}

const electionData = {
  "BYE 2019-202X": {election: ByElection2019, boundary: WestminsterConst},
  "GE2019":        {election: Election2019,   boundary: WestminsterConst},
  "BYE 2017-2015": {election: ByElection2017, boundary: WestminsterConst},
  "GE2017":        {election: Election2017,   boundary: WestminsterConst},
  "BYE 2015-2017": {election: ByElection2015, boundary: WestminsterConst},
  "GE2015":        {election: Election2015,   boundary: WestminsterConst}
};

/**
 * Event Handler for changing the active constituency to display results for
 */
function changeFocus (newFocus) {
  state.focusOn = newFocus;
}

onMounted(async () => {
  WestminsterConst.value = await d3.json("./WestminsterConst.geojson");
  Election2019.value = await d3.json("./2019.json");
  Election2017.value = await d3.json("./2017.json");
  Election2015.value = await d3.json("./2015.json");

  ByElection2019.value = await d3.json("./2019-_ByElections.json");
  ByElection2017.value = await d3.json("./2017-2019_ByElections.json");
  ByElection2015.value = await d3.json("./2015-2017_ByElections.json");
});

const dataSrc = computed(() => {
  return electionData[selectedElectionData.value];
})

const focusResult = computed(() => {
  return dataSrc.value.election.value[state.focusOn] || {};
});

</script> 

<style>

</style>
