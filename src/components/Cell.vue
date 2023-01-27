<template>
    <div v-if="props.tileValue === null" class="cell">{{ props.nr }}</div>
    <div v-else class="cell">
        <div class="tile" :class="bgColorMatchingTileValue">{{ props.tileValue }}</div>
    </div>
</template>

<script setup lang="ts">
import { computed } from "vue";

const props = defineProps<{
  nr: number,
  tileValue: number | null,
  isRandomlyPlaced?: boolean
}>();

const bgColorMatchingTileValue = computed(() => {
    let cssClasses = `background-color-${props.tileValue}`;
    if (props.isRandomlyPlaced === true) {
        cssClasses = cssClasses.concat(" randomly-placed");
    }
    return cssClasses;
});

</script>

<style scoped>
.cell {
    background-color: lightgray;
    border-radius: 1vmin;
    color: black;
    font-size: 3vmin;
    position: relative;
}

.tile {
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 1vmin;
    font-size: 5vmin;
    font-weight: bold;
    position: absolute;
    min-height: 20vmin;
    min-width: 20vmin;
    border: 3px solid black;

    --bg-color-2: #99FFFF;
    --bg-color-4: #99FFCC;
    --bg-color-8: #99FF99;
    --bg-color-16: #99FF66;
    --bg-color-32: #99FF33;
    --bg-color-64: #99FF00;
    --bg-color-128: #99CC00;
    --bg-color-256: #999900;
    --bg-color-512: #996600;
    --bg-color-1024: #993300;
    --bg-color-2048: #990000;
}

.background-color-2 {
    background-color: var(--bg-color-2);
}
.background-color-4 {
    background-color: var(--bg-color-4);
}
.background-color-8 {
    background-color: var(--bg-color-8);
}
.background-color-16 {
    background-color: var(--bg-color-16);
}
.background-color-32 {
    background-color: var(--bg-color-32);
}
.background-color-64 {
    background-color: var(--bg-color-64);
}
.background-color-128 {
    background-color: var(--bg-color-128);
}
.background-color-256 {
    background-color: var(--bg-color-256);
}
.background-color-512 {
    background-color: var(--bg-color-256);
}
.background-color-1024 {
    background-color: var(--bg-color-256);
}
.background-color-2048 {
    background-color: var(--bg-color-2048);
    animation: opacityFade 1s infinite;
}

@keyframes opacityFade { 
    from {
        opacity: 0.5;
    } 
    to {
        opacity: 1
    }
}

.randomly-placed {
    animation: tileGrow 0.3s ease-in;
}

@keyframes tileGrow {
    from {
        transform: scale(0);
    }
    to {
        transform: scale(1);
    }
}
</style>