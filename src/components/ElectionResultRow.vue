<template>
    <div class="flex items-center rounded-xl bg-slate-800 bg-opacity-50 my-2 px-2 py-1">
        <span class="basis-2/6">
            {{props.candidate.candidate}} (<span class="font-bold" :style="'color: ' + getPartyColour(props.candidate.party)">{{props.candidate.party}}</span>)
        </span>
        <span class="basis-2/6">
            {{props.candidate.votes}} ({{ Math.floor((props.candidate.votes / props.totalVotes * 100) * 100) / 100 }}%)
        </span>
        <div class="basis-2/6">
            <ProgressBar 
                :width="`${100 * props.candidate.votes / props.totalVotes}%`" 
                :colour="getPartyColour(props.candidate.party)">
            </ProgressBar>
            <span class="float-right">
                {{props.candidate.change}}
            </span>
        </div>
    </div>
</template>
<script setup>
import ProgressBar from './ProgressBar.vue';
import parties from "../assets/parties.json";

const props = defineProps({
    "candidate": Object,
    "totalVotes": Number
});

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