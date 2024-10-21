<script setup lang="ts">
import { onMounted, ref, shallowRef } from 'vue'
import Map from 'ol/Map'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
import { Feature } from 'ol'
import Point from 'ol/geom/Point'
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import { Icon, Style } from 'ol/style'
import { defaults } from 'ol/control/defaults'
import { fromLonLat } from 'ol/proj'
// type Props = {}
// type Emits = {
//   (e: 'click', value: any): void
// }

// const props = defineProps<Props>()
// const emits = defineEmits<Emits>()

const refMap = ref()
const mapInstance = shallowRef<Map>()

// function getCurrentLocation(): void {
//   if ('geolocation' in navigator) {
//     navigator.geolocation.getCurrentPosition(successCallback, errorCallback)
//   } else {
//     console.error('Geolocation is not supported by this browser.')
//   }
// }

const getCurrentLocation = () =>
  new Promise<GeolocationPosition>((resolve, reject) => {
    if ('geolocation' in navigator) {
      navigator.geolocation.getCurrentPosition(
        (position: GeolocationPosition) => {
          resolve(position)
        },
        (error: GeolocationPositionError) => {
          switch (error.code) {
            case error.PERMISSION_DENIED:
              reject('User denied the request for Geolocation.')
              break
            case error.POSITION_UNAVAILABLE:
              reject('Location information is unavailable.')
              break
            case error.TIMEOUT:
              reject('The request to get user location timed out.')
              break
            default:
              reject('An unknown error occurred.')
              break
          }
        },
      )
    }

    reject('Geolocation is not supported by this browser.')
  })

onMounted(async () => {
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
      zoom: 16,
      constrainResolution: true,
    }),
  })

  let latitude = 55.791977
  let longitude = 37.603244

  try {
    const position = await getCurrentLocation() // 55.791977, 37.603244
    console.log('position', { position })
    latitude = position.coords.latitude
    longitude = position.coords.longitude
  } catch (err) {}

  const markerCoordinates = fromLonLat([longitude, latitude])
  mapInstance.value.getView().setCenter(markerCoordinates)

  const marker = new Feature({
    geometry: new Point(markerCoordinates),
  })

  marker.setStyle(
    new Style({
      image: new Icon({
        anchor: [0.5, 1],
        src: 'img/pin.png', // Replace with your icon path
        // scale: 0.1, // Adjust size as needed
      }),
    }),
  )

  // Create a vector source and layer for the marker
  const vectorSource = new VectorSource({
    features: [marker],
  })

  const vectorLayer = new VectorLayer({
    source: vectorSource,
  })

  mapInstance.value.addLayer(vectorLayer)
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
