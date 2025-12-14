<script setup lang="ts">
import { ref, watch, PropType } from "vue"
import BitVisualizer from './BitVisualizer.vue';
import ClusterSlider from './ClusterSlider.vue';
import { SimilarityGraph, buildSimilarityGraph, clusterWithCohesion, mergeVectors } from "@andorsearch/qry-codes"

export interface Cluster {
  indices: number[]
  mergedVector: Uint8Array,
  score: number
}

const emit = defineEmits<{
  /**
   * Called whenever the similarity graph is first built.
   */

  similiarityGraphBuilt: [simGraph: SimilarityGraph]
}>()

const props = defineProps({
  /**
    * An array of vector embeddings, each expressed as a Uint8Array.
    * These must all be of the same length i.e. the same number of dimensions.
  */
  vectors: {
    type: Array<Uint8Array>,
    required: true
  },
  /**
    * The minimum similarity connecting all vectors in a cluster.
    * In practice acceptable values range from 0.5 to 1.
    * A binary vector resembling random white noise (think of an untuned TV) would 
    * match 50% of bits in typical vectors. That is why no-similarity score around 0.5
    * while an exact match is 1.
  */
  minClusterSim: {
    type: Number,
    default: 0.7
  },
  /**
    * Filters clusters that have less than this number of vectors in it
  */
  minDocsPerCluster: {
    type: Number,
    default: 1
  },
  /**
    * The css class given to each cluster element
  */
  clusterClass: {
    type: String,
    default: "aos-qry-codes-cluster"
  },
  /**
    * The css class given to the top-level element that contains all clusters
  */
  clusterContainerClass: {
    type: String,
    default: "aos-qry-codes-clusters-container"
  },
  /**
   * The location of the slider used to change minClusterSim and therefore the number of clusters formed.
   * @values none, top, bottom
   */
  slider: {
    // type: String as () => "none" | "top" | "bottom", 
    type: String,
    default: "top"
  },
  /**
   * An optional function to sort the list of clusters before they are displayed left to right (highest score first).
   * The default sort order is by cluster size (the number of similar vector elements in each cluster)
   */
  clusterScorer: {
    type: Function,
    required: false,
    default: (cluster: Cluster) => cluster.indices.length  // Default no-op sort function (keeps original order)
  },
    /**
      * Function called to cluster vectors
    */
    clusteringAlgo: {
        type: Function as PropType<(simGraph: SimilarityGraph, similarityThreshold:number) => number[][]>,
        required: false,
        default: clusterWithCohesion
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
    emit('similiarityGraphBuilt', simGraph)
  }
  clusters.value = []
  let unsortedClusters: Cluster[] = []
  let clusteredIDs: number[][] = props.clusteringAlgo(simGraph, sliderSim.value)
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

    <div :class="clusterContainerClass" >
      <div v-for="cluster, clusterIndex in clusters" :class="props.clusterClass">

        <!-- @slot Slot used for the top element for each cluster
            @binding {number[]} clusterVectorIndices The indices of each vector element grouped in this cluster
            @binding {Uint8Array} clusterMergedVector The averaged vector of all the vectors in this cluster (useful for querying vector databases for more similar content)
            @binding {number} clusterIndex The index of the cluster in the list of clusters (zero based)
        -->
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

        <div class="aos-cluster-contents" >
          <div v-for="vectorIndex, index in cluster.indices">
            <!-- @slot Slot used for each vector element arranged vertically in a cluster
                @binding {number} elementNumber The index of the vector element in this cluster (zero based)
                @binding {number} vectorIndex The index of the original vector in the "vectors" array passed in properties.
                @binding {number} clusterIndex The index of the cluster in the list of displayed clusters
                @binding {number} clusterSize The number of similar vectors grouped in this cluster
            -->
            <slot name="clusterElement" :vectorIndex="vectorIndex" , :elementNumber="index" :clusterIndex="clusterIndex"
              :clusterSize="cluster.indices.length">
              Result #{{ vectorIndex }}-{{ index }}
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

<style>
.aos-qry-codes-clusters-container {
  display:flex;
  flex-direction: row;
  overflow-x: scroll;
  overflow-y: clip;
  width: 100vw;
  gap:5px;
  height:300px;
}


.aos-qry-codes-cluster {
  font-size: x-small;
  border-radius: 5px;
  padding: 5px;
  max-width: 200px;
  border-style: solid;
  border-color: #f1f1f1;
  display: flex;
  flex-direction: column;
}

.aos-cluster-contents {
    margin-top: 6px;    
    gap: 10px;
    overflow-y: scroll;
    display: flex;
    flex-direction: column;
}

/* .aos-qry-codes-cluster:hover {
  background-color: #eaeaea;
} */
</style>
