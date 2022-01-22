<template>

<datalist id="selectionList">
    <option 
        v-for="key of Object.keys(props.election.value)" 
        :key="key" 
        :value="props.election.value[key].id"
    >{{ props.election.value[key].name }}
    </option>

</datalist>

</template>
<script setup>

import { watch } from 'vue';

const props = defineProps({
    "election": Object
});

watch(props.election, () => {
    console.log("election", props.election.value);
});

</script>