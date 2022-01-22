<template>
    <div class="w-full h-full rounded-lg shadow-lg bg-slate-600">
        <svg ref="map" class="rounded-lg"></svg>
    </div>
</template>
<script setup>

import { ref, onMounted, watch, watchEffect } from "@vue/runtime-core";
import * as d3 from "d3";
import parties from "../assets/parties.json";

import { pSBC } from "../js/pSBC";

const props = defineProps({
    "boundary": Object,
    "election": Object,
    "displayType": Object,
    "tagProperty": String
});

const emit = defineEmits(["changeFocus"]);

const height = 600, width = 700;
const zoom = d3.zoom();

var map;
var zoomLayer;

var selectedPath = null;
var mouseLocked = false;

function getPartyColour (partyCode) {

    if (partyCode == undefined)
        return "#616161";

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
function redrawGeoMap (boundary) {

    console.log("boundary", boundary);
    if (boundary == undefined || boundary == {} || boundary.features == undefined || boundary.features.length == 0)
        return;

    let geoGenerator = d3.geoPath()
        .projection(d3.geoMercator().fitExtent([[0, 0], [width, height]], boundary));

    map.attr("viewBox", [[0, 0], [width, height]]);
    map.call(
        zoom.on("zoom", (e) => {
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
            .attr("data-tag", (feature) => feature.properties[props.tagProperty])
            .attr("fill", "red")
            .on("mouseover", (event) => { // Show seat data on mouseover
                if (mouseLocked)
                    return;
                else
                    emit("changeFocus", event.target.getAttribute("data-tag"));
            })
            .on("click", (event) => { // Zoom into selected map part
                let bbox = event.target.getBBox();
                let [centreX, centreY] = [bbox.x + (bbox.width / 2), bbox.y + (bbox.height / 2)];
                
                if (selectedPath) {
                    selectedPath.setAttribute("stroke", "");

                    // Zoom out if clicking on the same constituency
                    if (selectedPath.getAttribute("data-tag") == event.target.getAttribute("data-tag")) {
                        map.transition().duration(500).call(zoom.transform, d3.zoomIdentity);

                        mouseLocked = false;
                        selectedPath = null;
                        return;
                    }
                }

                map.transition().duration(500).call(
                    zoom.transform, 
                    d3.zoomIdentity
                        .translate(width / 2, height / 2)
                        .scale(8)
                        .translate(-centreX, -centreY)
                    );
                event.target.setAttribute("stroke", "gold");
                event.target.setAttribute("stroke-width", "0.05%");
                event.target.setAttribute("stroke-linejoin", "round");

                d3.select(event.target).raise(); // Move to front

                mouseLocked = true;
                selectedPath = event.target;
                emit("changeFocus", event.target.getAttribute("data-tag"))
            });
}

/**
 * Colour the SVG elements by election winners
 */
function recolourGeoMap (electionData, display) {

    zoomLayer.selectAll("path").attr("fill", (path) => {
        let filterFunc;
        switch (props.displayType.filter) {
            case "majority": filterFunc = filterMajorityOnly; break;
            case "plurality": filterFunc = filterPluralityOnly; break;
            default:
                filterFunc = filterAll; 
        }

        if (!filterFunc(electionData, path))
            return "#616161";

        switch (props.displayType.fill) {
            case "majority": return colourByMajority(electionData, path);
            case "second": return colourBySecondParty(electionData, path);
            case "turnout": return colourByTurnout(electionData, path);
            default:
                return colourByWinningParty(electionData, path);
        }
    });
}

function filterMajorityOnly (electionData, path) {
    let res = electionData[path.properties[props.tagProperty]];
    
    if (!res || !res.candidates[0]) return false;
    if (res.turnout <= 0) return false;

    return (res.candidates[0].votes / res.turnout > 0.5)
}

function filterPluralityOnly (electionData, path) {
    return !filterMajorityOnly(electionData, path);
}

function filterAll () {
    return true;
}

function colourByWinningParty (electionData, result) {
    let winningParty = electionData[result.properties[props.tagProperty]]?.mp.party || undefined;
    return getPartyColour(winningParty);
}

function colourBySecondParty (electionData, result) {
    let secondParty = electionData[result.properties[props.tagProperty]]?.candidates[1]?.party || undefined;
    return getPartyColour(secondParty);
}

function colourByTurnout (electionData, result) {
    let averageTurnout = 0.5; // Replace with average turnout expr this.seatData[this.currentMap.year].turnout;
    let res = electionData[result.properties[props.tagProperty]];
    if (!res) {
        return "#616161";
    }
    return pSBC(((1 - averageTurnout) - (res.turnout / res.electorate)) * 1.5, "#aed581", false, true);
}

function colourByMajority (electionData, result) {
    let res = electionData[result.properties[props.tagProperty]];
    if (!res) {
        return "#616161";
    }
    let winningParty = electionData[result.properties[props.tagProperty]]?.mp.party || undefined;
    return pSBC(
        (1 - (res.majority / res.candidates[0].votes)) - 0.25,
        getPartyColour(winningParty),
        false, 
        true
    );
}


/**
 * Create instances of d3 and do initial stuff when this component is mounted into the document
 */
onMounted(async () => {

    map = d3.select("div svg")
        .attr("viewBox", [[0, 0], [width, height]]);
    zoomLayer = map.append("g");

    watchEffect(() => {
        redrawGeoMap(props.boundary.value);
    });

    watchEffect(() => {
        console.log("recolouring map");
        recolourGeoMap(props.election.value);
    });

});



</script>