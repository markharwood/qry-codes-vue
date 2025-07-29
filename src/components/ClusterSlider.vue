<script setup lang="ts">
import { ref, watch, onMounted } from "vue";

const props = defineProps({
    /**
      * The minimum similarity connecting all vectors in a cluster.
      * In practice acceptable values range from 0.5 to 1.
      * A binary vector resembling random white noise (think of an untuned TV) would 
      * match 50% of bits in typical vectors. The mininum value of 0.5 therefore typically
      * links all provided vectors into a single cluster
    */
    min: {
        type: Number,
        default: 0.5,
    },
    /**
      * The maximum similarity connecting all vectors in a cluster.
      * A value of 1 typically represents identical content and is there the max similarity
    */
    max: {
        type: Number,
        default: 0.99,
    },
    /**
     * The property which will hold the value of the slider's current position
     */
    modelValue: {
        type: Number,
        required: true,
    },
    /**
     * The size in the pixels of the thumb track shown in the slider - larger values help on mobile devices
     */
     thumbSize: {
        type: Number,
        default: 18, // Default thumb size in pixels
    }
});

const emit = defineEmits<{
    /** @ignore */
    mounted: [];
    /** @ignore */
    "update:modelValue": [newValue: number];
}>();

const val = ref(props.modelValue);

watch(
    () => props.modelValue,
    (newValue) => {
        val.value = newValue;
    }
);

onMounted(() => {
    emit("mounted");
});

function changeVal(event: Event) {
    const newValue = parseFloat((event.target as HTMLInputElement).value);
    emit("update:modelValue", newValue);
}
</script>

<template>
    <div class="slider-container">
        <!-- @slot Slot used for the description shown above the slider input
        -->        
        <slot name="header">
            <div class="slider-label">Topic scope</div>
        </slot>
        <div class="slider-wrapper">
            <!-- @slot Slot used for the description shown to the left of the slider input to indicate low-similarity clustering
            -->        
            <slot name="leftLabel">
                <span>broad ←</span>
            </slot>
            <input type="range" v-model="val" :min="props.min" :max="props.max" step="0.01" class="custom-slider"
                @input="changeVal" />
            <!-- @slot Slot used for the description shown to the right of the slider input to indicate high-similarity clustering
            -->        
            <slot name="rightLabel">
                <span>→ specific</span>
            </slot>
        </div>
    </div>
</template>

<style scoped>
.slider-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: left;
    margin-top: 5px;
    font-size: 12px;
}

.slider-label {
    display: flex;
    justify-content: center;
    font-weight: bold;
}

.slider-wrapper {
    display: flex;
    align-items: center;
}

.custom-slider {
    -webkit-appearance: none;
    width: 150px;
    height: 6px;
    background: #e4e7ed;
    border-radius: 3px;
    outline: none;
}

.custom-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: v-bind(props.thumbSize+"px");
    height: v-bind(props.thumbSize+"px");
    /* height: 12px; */

    background-color: white;
    border: 2px solid #409eff;

    /* background: #409eff; */
    border-radius: 50%;
    cursor: pointer;
}

.custom-slider::-moz-range-thumb {
    width: v-bind(props.thumbSize+"px");
    height: v-bind(props.thumbSize+"px");
    /* height: 12px; */
    /* background: #409eff; */

    background-color: white;
    border: 2px solid #409eff;

    border-radius: 50%;
    cursor: pointer;
}



.custom-slider:focus {
    background: #c6e2ff;
}
</style>