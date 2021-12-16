<template>
    <div class="w-full bg-slate-600 rounded-lg shadow-lg p-4 text-gray-100">
        
        <h1 v-if="!props.result.name" class="block font-bold text-2xl">Select or Mouseover A Constituency To Begin</h1>
        
        <h1 v-if="props.result.name" class="inline-block font-bold text-3xl">{{ props.result.name || "" }}</h1>
        <h5 v-if="props.result.id" class="inline-block font-thin text-sm pl-2">({{ props.result.id || "" }})</h5>

        <h3 v-if="props.result.mp" class="block font-bold text-xl">
            Elected MP: {{ props.result.mp.name || "N/A" }}
            (<span
                :style="'color: ' + getPartyColour(props.result.mp.party)"
            >{{ props.result.mp.party || "Ind" }}</span>)
        </h3>

        <h3
            v-if="props.result.result && props.result.majority && props.result.electorate"
            class="font-bold text-lg inline-block"
        >
            Result:
            <span class="font-normal">{{ props.result.result }}</span>
            majority of
            <span class="font-normal">{{ props.result.majority }} ({{ Math.floor((props.result.majority / props.result.electorate * 100) * 100) / 100 }}%)</span>
        </h3>

        <h3 v-if="props.result.electorate && props.result.turnout" class="font-bold text-lg block">
            Electorate:
            <span class="font-normal">{{ props.result.electorate }}</span>
            turnout of
            <span
                class="font-normal"
            >{{ props.result.turnout }} ({{ Math.floor((props.result.turnout / props.result.electorate * 100) * 100) / 100 }}%)</span>
        </h3>

        <div v-if="props.result.candidates">
            <ElectionResultRow 
                v-for="candidate in candidatesDisplay(props.result.candidates)[0]" 
                :key="candidate.candidate" :candidate="candidate" 
                :totalVotes="props.result.turnout">
            </ElectionResultRow>
            <span v-if="candidatesDisplay(props.result.candidates)[1].length > 0">
                Others: {{ candidatesDisplay(props.result.candidates)[1].join(", ") }}.
            </span>
        </div>
        
    </div>
</template>
<script setup>
import { computed } from '@vue/runtime-core';
import parties from "../assets/parties.json";
import ProgressBar from './ProgressBar.vue';
import ElectionResultRow from './ElectionResultRow.vue';

const props = defineProps({
    result: Object
});

function candidatesDisplay (candidates) {
    let threshold = 5;
    let counter = 0;
    let output = [];

    let otherParties = [];
    let otherVotes = 0;
    for (let candidate of candidates) {

        if (counter < threshold || (counter == threshold && counter == candidates.length - 1)) {
            output.push(candidate);
        } else {
            otherVotes += candidate.votes;
            otherParties.push(candidate.party);
        }

        counter++;
    }

    if (otherVotes != 0) {
        output.push({"candidate": "Other", "party": "Various", "votes": otherVotes});
    }

    return [output, otherParties];
}

function getPartyColour(partyCode) {
    for (let party of parties.parties) {
        if (party.name == partyCode ||
            party.fullname == partyCode ||
            party.abbr.includes(partyCode) ||
            party.abbr.includes(partyCode.toLowerCase())
        ) {
            return party.colour;
        }
    }

    return "#DDDDDD";
}
</script>