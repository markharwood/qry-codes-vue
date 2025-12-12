<template>
    <div class="qrycode-chip">
        <div class="lens">
            <div class="lens-inner">
                <BitVisualizer :data="visibleBytes" :cols="14" :cellSize="2" :pixelColor="props.pixelColor" />
            </div>

            <!-- Optional aggregation badge (does NOT zoom) -->
            <div v-if="badgeText" class="badge" aria-hidden="true">
                <span class="badge-pill">{{ badgeText }}</span>
            </div>
        </div>
        <div class="handle" />
    </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import BitVisualizer from './BitVisualizer.vue'


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
    },

    /**
     * Optional label text (e.g. number of fused elements). If provided, a small
     * label is shown inside the lens. Text does NOT scale on hover.
     */
    label: {
        type: [String, Number],
        default: undefined
    }
})

// Use only first 24 bytes (24*8=192 pixels)
const visibleBytes = computed(() => props.data.slice(0, 24))

const badgeText = computed(() => {
    const b = props.label
    if (b === undefined || b === null) return ''
    const s = String(b).trim()
    return s.length ? s : ''
})
</script>

<style scoped>
.qrycode-chip {
    position: relative;
    display: inline-block;
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

/* Contents that gets magnified */
.lens-inner {
    transform-origin: center center;
    transition: transform 0.15s ease-out;
}

/* Optional badge overlay (does NOT zoom) */
.badge {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    pointer-events: none;
}

.badge-pill {
    /* ...your existing styles... */
    background: rgba(255, 255, 255, 0.15);
    font-size: 14px;
    font-weight: bolder;
    color: #000000;

    -webkit-text-stroke: 1.2px rgba(255, 255, 255, 0.95);
    paint-order: stroke fill;
    /* helps in some engines */

    /* fallback halo for browsers that ignore text-stroke */
    text-shadow:
        0 1px 0 rgba(255, 255, 255, 0.7),
        1px 0 0 rgba(255, 255, 255, 0.7),
        0 -1px 0 rgba(255, 255, 255, 0.7),
        -1px 0 0 rgba(255, 255, 255, 0.7);
}


.badge::before {
    content: "";
    position: absolute;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: rgba(70, 100, 255, 0.85);
    /* tweak to taste */
    box-shadow: 0 0 0 2px rgba(255, 255, 255, 0.35);
    /* subtle rim */
    z-index: -1;
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

/* Hover: magnifies the contents inside lens (badge stays static) */
.qrycode-chip:hover .lens-inner {
    transform: scale(1.2);
}
</style>