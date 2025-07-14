# QRy-codes Vue Components

Vue 3 component for visualizing [qry-codes](http://qry.codes) and clustering them.
Useful for search applications built using BinaryVectors.

## Installation

```sh
npm install @andorsearch/qry-codes-vue@next
```


## Components
* [QryCluster](https://github.com/markharwood/qry-codes-vue/blob/main/docs/src/components/QryCluster.md) for clustering similar vectors interactively with a similarity threshold slider
* [BitVisualizer](https://github.com/markharwood/qry-codes-vue/blob/main/docs/src/components/BitVisualizer.md) for showing vectors as QRy codes (QR-like codes)


## Usage

Check out the "Hello World" demo at https://github.com/markharwood/qry-codes-vue

Applications will typically load data in JSON form with the binary embeddings encoded
as hex or base64 strings. These need to be converted into Uint8Array objects for use
in clustering algorithms. The 'bytesConversion' object offers "fromBase64" and "fromHex"
helper functions to aid in this:


```typescript
import { bytesConversion } from "@andorsearch/qry-codes"
let myDocs = ref<any[]>([])

async function loadData() {
  const data = await fetch("/mydata/myDocs.json");
  const json = await data.json()
  json.forEach(doc => {
    // Replace simple string objects loaded in json with Uint8Array binary vectors
    doc["embedding"] = bytesConversion.fromBase64(doc["embedding"])
    myDocs.value.push(doc)
  })
}
onMounted(() => {
  loadData()
});
```
Once loaded, the data can be passed to the QryCluster component for clustering and visualization.
Clusters are listed horizontally and within each cluster a number of vectors are listed vertically
where typically applications will show original text and/or images that were originally used to 
create the vectors rather than the raw vector data.     


```typescript
import { QryCluster, BitVisualizer } from '@andorsearch/qry-codes-vue'

    <QryCluster v-if="myDocs.length > 0" :vectors="myDocs.map(doc => doc['embedding'])" :minDocsPerCluster="1">
      <!-- This slot allows the header of each cluster to be rendered. 
          The 'clusterMergedVector' is the average UInt8Array of all elements in the cluster.
          The 'clusterVectorIndices' is an array of numbers representing the elements in the cluster where each
          number is the index of the vector passed in the "vectors" property above.
          Below we use a BitVisualizer to show the clusterMergedVector property as a set of pixels or "QRy code".
        -->
      <template #clusterHeader="{ clusterMergedVector, clusterVectorIndices }">
        <BitVisualizer :data="clusterMergedVector" :cols="64" :cellSize="1" />
        {{ clusterVectorIndices.length }} matches
      </template>

      <!-- This slot allows the content of each element in a cluster to be rendered. 
          The 'vectorIndex' is the index of the vector passed in the "vectors" property above.
        -->
      <template #clusterElement="{ vectorIndex }">
        <div class="article">
          {{ myDocs[vectorIndex]['headline'] }}
        </div>
      </template>
    </QryCluster>

```





