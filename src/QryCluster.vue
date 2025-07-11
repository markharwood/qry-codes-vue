<script setup lang="ts">
import { ref, watch } from "vue"
import BitVisualizer from './BitVisualizer.vue';
import ClusterSlider from './ClusterSlider.vue';
import { SimilarityGraph, buildSimilarityGraph, clusterByThreshold, mergeVectors } from "@andorsearch/qry-codes"

export interface Cluster {
  indices: number[]
  mergedVector: Uint8Array,
  score: number
}

const props = defineProps({
  vectors: {
    type: Array<Uint8Array>,
    required: true
  },
  minClusterSim: {
    type: Number,
    default: 0.7
  },
  vw: {
    type: Number,
    default: 100
  },
  minDocsPerCluster: {
    type: Number,
    default: 2
  },
  clusterClass: {
    type: String,
    default: "qry-codes-cluster"
  },
  slider: {
    // type: String as () => "none" | "top" | "bottom", 
    type: String,
    default: "top"
  },
  clusterScorer: {
    type: Function,
    required: false,
    default: (cluster: Cluster) => cluster.indices.length  // Default no-op sort function (keeps original order)
  }
})
let minFoundSim = ref(0)
let maxFoundSim = ref(1)

let sliderSim = ref(props.minClusterSim)

let clusters = ref<any[]>([])
let simGraph: SimilarityGraph | undefined = undefined
clusterResults()

watch(sliderSim, () => {
  if (sliderSim.value != props.minClusterSim) {
    clusterResults()
  }
})

watch(
  () => props.minClusterSim,
  (newValue) => {
    if (sliderSim.value != newValue) {
      sliderSim.value = newValue;
      clusterResults()
    }
  }
);


function clusterResults() {
  let max = 0
  let min = 1
  if (simGraph == undefined) {
    simGraph = buildSimilarityGraph(props.vectors)
    simGraph.edgeScores.forEach(cluster => {
      cluster.forEach(score => {
        max = Math.max(score, max)
        min = Math.min(score, min)
      })
    })
    minFoundSim.value = min
    maxFoundSim.value = max
  }
  clusters.value = []
  let unsortedClusters: Cluster[] = []
  let clusteredIDs: number[][] = clusterByThreshold(simGraph, sliderSim.value)
  clusteredIDs = clusteredIDs.filter(cluster => cluster.length >= props.minDocsPerCluster)
  clusteredIDs.forEach((cluster) => {
    // Compute the average for each cluster
    const clusterEmbeddings: Uint8Array[] = []
    cluster.forEach(hitID => {
      clusterEmbeddings.push(props.vectors[hitID])
    })
    let islandAverage = mergeVectors(clusterEmbeddings)

    let newCluster: Cluster = {
      indices: cluster,
      mergedVector: islandAverage,
      score: 0
    }
    // Compute a score for this cluster (default is by cluster size - user can override)
    newCluster.score = props.clusterScorer(newCluster)
    unsortedClusters.push(newCluster)
  })
  clusters.value = unsortedClusters.sort((a, b) => b.score - a.score);
}

</script>

<template>
  <div v-if="clusters">
    <div v-if="props.slider == 'top'" style="padding-bottom: 10px;">
      <ClusterSlider v-model="sliderSim" :min="minFoundSim" :max="maxFoundSim"></ClusterSlider>
    </div>

    <div :style="'display:flex;flex-direction: row;overflow-x: scroll;width: ' + props.vw + 'vw;gap:5px;height:300px;'">
      <div v-for="cluster, clusterIndex in clusters" :class="props.clusterClass">

        <slot name="clusterHeader" :clusterVectorIndices="cluster.indices" :clusterMergedVector="cluster.mergedVector"
          :clusterIndex="clusterIndex">
          <div>
            <BitVisualizer :data="cluster.mergedVector" :cols="64" :cellSize="1" />
            <div v-if="cluster.indices.length > 1" style="text-align: center;">
              {{ cluster.indices.length }} matches
            </div>
            <div v-if="cluster.indices.length == 1" style="text-align: center;">
              1 match
            </div>
          </div>
        </slot>

        <div style="display: flex;flex-direction: column;gap:10px;max-height:230px;overflow-y: scroll;margin-top: 6px;">
          <div v-for="vectorIndex, index in cluster.indices">
            <slot name="clusterElement" :vectorIndex="vectorIndex" , :elementNumber="index"
              :clusterSize="cluster.indices.length">
              Result #{{ docId }}-{{ index }}
            </slot>
          </div>
        </div>
      </div>

    </div>
    <div v-if="props.slider == 'bottom'">
      <ClusterSlider v-model="sliderSim" :min="minFoundSim" :max="maxFoundSim"></ClusterSlider>
    </div>

  </div>

</template>

<style scoped>
.qry-codes-cluster {
  font-size: x-small;
  border-radius: 5px;
  text-align: left;
  padding: 10px;
  max-width: 200px;
  border-style: solid;
  border-color: #f1f1f1;
}

.qry-codes-cluster:hover {
  background-color: #eaeaea;
}
</style>
