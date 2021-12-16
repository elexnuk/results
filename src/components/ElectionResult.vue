<template>

    <div class="w-full bg-slate-300 rounded-lg shadow-lg p-4">
        
        <h1 class="inline-block font-bold text-xl">{{ props.result.name || "" }}</h1> 
        <h5 class="inline-block font-bold text-sm pl-2">[{{ props.result.id || "" }}]</h5>

        <span class="font-mono text-sm block" >{{ JSON.stringify(props.result) }} </span>
    
    </div>

</template>
<script setup>
    import { onMounted, watch } from '@vue/runtime-core';

    const props = defineProps({
       result: Object
    });

    onMounted(() => {
        console.log(props.result);
    });

    watch(props.result, () => {
        console.log("New result ", props.result);
    })
</script>