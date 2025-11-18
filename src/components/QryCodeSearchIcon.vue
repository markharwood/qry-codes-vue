<template>
    <div class="qrycode-chip">
        <div class="lens">
            <div class="lens-inner">
                <BitVisualizer :data="visibleBytes" :cols="14" :cellSize="2" :pixelColor="props.pixelColor" />
            </div>
        </div>
        <div class="handle" />
    </div>
</template>

<script setup lang="ts">
import BitVisualizer from './BitVisualizer.vue';

const props = defineProps({

  data: {
  /**
    * The binary vector embedding (aka QRy code) that we want to visualize in the magnifying glass
  */    
    type: Uint8Array,
    required: true
  },

  /**
    * The normal color of bits set to 1 in the binary embedding
  */    
  pixelColor: {
    type: String,
    default: '#aaa9e8'
  }

})
const visibleBytes = props.data.slice(0,  24) //Use only 24 bytes (24*8=192 pixels)

</script>

<style scoped>
.qrycode-chip {
    position: relative;
    border: none;
    padding: 0;
    background: transparent;
    cursor: pointer;
}

/* circular “lens” */
.lens {
    position: relative;
    width: 28px;
    height: 28px;
    border-radius: 50%;
    overflow: hidden;
    border: 2px solid rgb(0 0 0 / 35%);
    display: flex;
    align-items: center;
    justify-content: center;
}


/* GLASS HIGHLIGHT Radial “reflection” - suspected added lag on iphone for minimal aesthetic improvement */
/* .lens::after {
    content: "";
    position: absolute;
    inset: 0;
    border-radius: 50%;
    pointer-events: none;
    background: radial-gradient(circle at 20% 20%,
            rgba(255, 255, 255, 0.8),
            rgba(255, 255, 255, 0.0) 60%);
    mix-blend-mode: screen; 
} */


.qrycode-chip:hover .lens::after {
    opacity: 1;
    transform: scale(1.02);
}

/* diagonal “handle” */
.handle {
    position: absolute;
    width: 11px;
    height: 4px;
    border-bottom-right-radius: 2px;
    border-top-right-radius: 2px;
    background: rgb(0 0 0 / 35%);
    transform: rotate(40deg);
    bottom: 0px;
    right: -5px;
}

/* Hover: magnifies the contents inside lens */
.qrycode-chip:hover .lens-inner {
  transform: scale(1.2);  
}
</style>