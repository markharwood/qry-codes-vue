<script setup lang="ts">
import { ref, onMounted , watch} from "vue";
import { BitPatternCanvas } from "@andorsearch/qry-codes"

const props = defineProps({
  data: {
    type: Uint8Array,
    required: true
  },
  cols: {
    type: Number,
    default: 8
  },
  cellSize: {
    type: Number,
    default: 1
  },
  // If similarity >0 then we display pixels with colors that can represent
  // a bar chart indicating number of matching pixels
  barSimilarity: {
    type: Number,
    default: 0
  },
  pixelColor: {
    type: String,
    default: '#3937f18c'
  },
  noMatchPixelColor: {
    type: String,
    default: 'lightgray'
  },
  backgroundColor: {
    type: String,
    default: 'white'
  }

})

// Watch all props dynamically for any change
watch(
  props,
  (_newProps:any, _oldProps:any) => {
      redraw()
  },
  { deep: false } 
);




let canvas = ref()

let visualizer: BitPatternCanvas | undefined
function drawBitPattern() {
  visualizer = new BitPatternCanvas(canvas.value, props.data, getOptions());
}

function getOptions(): any {
  return {
    cols: props.cols,
    cellSize: props.cellSize,
    pixelColor: props.pixelColor,
    noMatchPixelColor: props.noMatchPixelColor,
    backgroundColor: props.backgroundColor,
    barSimilarity: props.barSimilarity
  }
}

function redraw() {
  if (visualizer) {
    visualizer.setOptions(getOptions())
    visualizer.redraw(props.data)
  }
}
defineExpose({ redraw })

onMounted(() => {
  drawBitPattern()
})
</script>
<template>
  <canvas ref="canvas"></canvas>
</template>
