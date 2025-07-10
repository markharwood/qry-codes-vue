<script setup lang="ts">
import { ref, watch, onMounted } from "vue";

const props = defineProps({
    min: {
        type: Number,
        default: 0.5,
    },
    max: {
        type: Number,
        default: 0.99,
    },
    modelValue: {
        type: Number,
        required: true,
    },
    thumbSize: {
        type: Number,
        default: 18, // Default thumb size in pixels
    }
});

const emit = defineEmits<{
    mounted: [];
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
        <slot name="header">
            <div class="slider-label">Topic scope</div>
        </slot>
        <div class="slider-wrapper">
            <slot name="leftLabel">
                <span>broad ←</span>
            </slot>
            <input type="range" v-model="val" :min="props.min" :max="props.max" step="0.01" class="custom-slider"
                size="small"
                @input="changeVal" />
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
    margin-top:5px;
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

/* Latest CGPT answers */

/* Style the thumb */
/* input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 40px; 
  height: 40px;
  background: #007aff;
  border-radius: 50%;
  cursor: pointer;
  position: relative;
} */

/* Prevent accidental taps on the track */
/* .custom-slider::-webkit-slider-runnable-track {
    width: 100%;
  height: 8px; 
  background: #ddd;
  border-radius: 4px;
} */



.custom-slider:focus {
    background: #c6e2ff;
}
</style>