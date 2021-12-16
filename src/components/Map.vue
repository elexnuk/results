<template>
    <div class="w-full h-full rounded-lg shadow-lg bg-slate-300">
        <svg ref="map"></svg>
    </div>
</template>
<script setup>

import { ref, onMounted, watch } from "@vue/runtime-core";
import * as d3 from "d3";
import parties from "../assets/parties.json";

const props = defineProps({
    "mapType": String,
    "dataSource": Object,
    "displayType": String
});

const emit = defineEmits(["changeFocus"]);

var map;
var zoomLayer;

const height = 600, width = 700;

function getPartyColour (partyCode) {
    for (let party of parties.parties) {
        if (party.name == partyCode || 
            party.fullname == partyCode || 
            party.abbr.includes(partyCode) || 
            party.abbr.includes(partyCode.toLowerCase())
        ) {
            return party.colour;
        }
    }

    return "#616161";
}

/**
 * Put boundary data into the svg element.
 */
function redrawGeoMap () {
    let boundary = props.dataSource.boundary.value;

    console.log("boundary", boundary);
    if (boundary == undefined || boundary == {} || boundary.features == undefined || boundary.features.length == 0)
        return;

    let geoGenerator = d3.geoPath()
        .projection(d3.geoMercator().fitExtent([[0, 0], [width, height]], boundary));

    map.attr("viewBox", [[0, 0], [width, height]]);
    map.call(
        d3.zoom().on("zoom", (e) => {
            zoomLayer.attr("transform", e.transform);
        })
        .scaleExtent([1, 8])
        .translateExtent([[0, 0], [width, height]])
    );
    zoomLayer.attr("transform", "translate(0 0) scale(1)");
    zoomLayer.selectAll("path")
        .data(boundary.features)
        .enter()
            .append("path")
            .attr("d", geoGenerator)
            .attr("data-tag", (feature) => feature.properties.PCON20CD)
            .attr("fill", "red")
            .on("mouseover", (event) => {
                emit("changeFocus", event.target.getAttribute("data-tag"));
            });
}

/**
 * Colour the SVG elements by election winners
 */
function recolourGeoMap () {
    let electionData = props.dataSource.election.value;

    zoomLayer.selectAll("path").attr("fill", (data) => {
        let winningParty = electionData[data.properties.PCON20CD].mp.party;
        return getPartyColour(winningParty);
    });
}

/**
 * Update the boundaries in the svg when the boundaries change
 */
watch(props.dataSource.boundary, () => {
    console.log("Src Boundaries changed ", props.dataSource.boundary.value);
    redrawGeoMap();
});

/**
 * Update the colouring when the election src changes.
 */
watch(props.dataSource.election, () => {
    console.log("Src Election changed ", props.dataSource.election.value);
    recolourGeoMap();
})

/**
 * Create instances of d3 and do initial stuff when this component is mounted into the document
 */
onMounted(async () => {
    console.log("Mounted with ", props.dataSource.boundary.value);
    map = d3.select("div svg")
        .attr("viewBox", [[0, 0], [width, height]]);
    zoomLayer = map.append("g");

    redrawGeoMap();
    recolourGeoMap();
});

</script>