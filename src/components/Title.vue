<template>
<!-- <a href="https://github.com/jamesm2w" class="" target="_blank">
<div
    class="m-4 mb-0 p-2 text-gradient text-transparent bg-clip-text w-min pointer-events-none select-none"
>
    <h3 class="font-mono font-bold text-3xl">jamesm2w/</h3>
    <h1 class="font-mono font-semibold text-5xl">
        Election
        <br />Results
    </h1>
</div>
</a> -->
<h1 class="text-5xl font-mono font-black text-gradient text-transparent bg-clip-text m-4 mb-0 p-2 pl-0 mt-3">
    Elections<br>
    <span class="text-3xl inline-block relative -top-5 font-semibold font-sans italic"><a href="https://elexn.uk/" target="_blank">elexn.uk</a></span><br>
</h1>
</template>
<script setup>

</script>
<style scoped>
.text-gradient {
    background-image: linear-gradient(
        -90deg,
        #8DC63F,
        #E4003B,
        #FAA61A,
        #0087DC
    );
    background-size: 400% 400%;
    animation: bg-gradient 10s ease infinite;
}

@keyframes bg-gradient {
    0% {
        background-position: 0% 50%;
    }
    50% {
        background-position: 100% 50%;
    }
    100% {
        background-position: 0% 50%;
    }
}
</style>