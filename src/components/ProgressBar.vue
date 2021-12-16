<template>
    <div 
        :style="'width: ' + props.width + '; background-color: ' + props.colour" 
        class="h-5 my-1 transition-all duration-500 ease-in float-left relative"
    ></div>
</template>
<script setup>

const props = defineProps({
    width: String,
    colour: String
});

</script>