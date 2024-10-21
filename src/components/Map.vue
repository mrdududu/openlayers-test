<script setup lang="ts">
import { onMounted, ref, shallowRef } from 'vue'
import Map from 'ol/Map'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
import { defaults } from 'ol/control/defaults'
// type Props = {}
// type Emits = {
//   (e: 'click', value: any): void
// }

// const props = defineProps<Props>()
// const emits = defineEmits<Emits>()

const refMap = ref()
const mapInstance = shallowRef<Map>()

onMounted(() => {
  mapInstance.value = new Map({
    target: refMap.value,
    controls: defaults({
      // zoom: false, // Disable default zoom controls
      // rotate: false,
      // attribution: false,
    }),
    layers: [
      new TileLayer({
        source: new OSM(),
      }),
    ],
    view: new View({
      center: [0, 0],
      zoom: 2,
      constrainResolution: true,
    }),
  })
})

defineExpose({ mapInstance })
</script>
<template>
  <div
    ref="refMap"
    class="ol-map-block"
    style="width: 100%; height: 100%"
  ></div>
</template>
<style scoped>
.ol-map-block {
  position: relative;
  width: 100%;
  height: 100%;
}
</style>
