<template>
<main style="height:100vh; width:100%">
    <l-map  ref="map" v-model:zoom="zoom" :center="[47.41322, -1.219482]" :use-global-leaflet="false">
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
        attribution=""
      ></l-tile-layer>
      <l-marker :lat-lng="[47.41322, -1.219482]"></l-marker>
      <l-control-scale></l-control-scale>
      <l-control-attribution :prefix="false" attribution="Demo"></l-control-attribution>
    </l-map>
</main>
</template>

<script setup>
import "leaflet/dist/leaflet.css";
import {  LMap, LTileLayer, LMarker, LControlScale, LControlAttribution } from "@vue-leaflet/vue-leaflet";
import { ref, useTemplateRef } from "vue";

const zoom = ref(2);
const map = useTemplateRef('map');
console.log(LMap)
</script>