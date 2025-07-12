<script setup lang="ts">
import { ref, onMounted , watch} from "vue";
import { BitPatternCanvas } from "@andorsearch/qry-codes"

const props = defineProps({

  data: {
  /**
    * The binary vector embedding (aka QRy code) that we want to visualize
  */    
    type: Uint8Array,
    required: true
  },
  /**
    * The number of bits to be rendered in each row (the number of rows is determined by the data length divided by
    * the number of columns)
  */    
  cols: {
    type: Number,
    default: 8
  },
  /**
    * The pixel size of each bit (values of less than one are accepted)
  */    
  cellSize: {
    type: Number,
    default: 1
  },
  /**
    * If barSimilarity >0 then we display pixels with colors that can represent
    * a bar chart indicating the number of matching bits
  */    
  barSimilarity: {
    type: Number,
    default: 0
  },
  /**
    * The normal color of bits set to 1 in the binary embedding
  */    
  pixelColor: {
    type: String,
    default: '#3937f18c'
  },
  /**
    * The color of bits set to 1 in the binary embedding where 'barSimilarity' is greater than zero
    * and the bit is non-matching.
    * Rather than rendering a true representation of which bits did not match in a search (this would
    * tend to blend colors) the bits are shown as a form of horizontal bar chart with all
    * 1 bits to the right of the bar end displayed in this "no match" color.
  */    
  noMatchPixelColor: {
    type: String,
    default: 'lightgray'
  },
  /**
    * The color of bits set to 0
  */    
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
