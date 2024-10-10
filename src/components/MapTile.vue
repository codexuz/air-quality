<template>
  <div class="map-wrap">
    <div class="map" ref="mapContainer"></div>
   
    <!-- AQI Legend -->
    <div class="legend">
      <div><span class="legend-color" style="background-color: #00E400;"></span> Yaxshi (0-50)</div>
      <div><span class="legend-color" style="background-color: #FFFF00;"></span> O'rtacha (51-100)</div>
      <div><span class="legend-color" style="background-color: #FF7E00;"></span> Ma'lum guruhlar uchun zararli (101-150)</div>
      <div><span class="legend-color" style="background-color: #FF0000;"></span> Zararli (151-200)</div>
      <div><span class="legend-color" style="background-color: #8F3F97;"></span> Juda Zararli (201-300)</div>
      <div><span class="legend-color" style="background-color: #7E0023;"></span> Zaharli (301-500)</div>
    </div>
  </div>
</template>

<script setup>
import { Map, MapStyle, config, Marker, Popup, MaptilerLogoControl } from '@maptiler/sdk';
import { WindLayer, TemperatureLayer, RadarLayer, PrecipitationLayer, ColorRamp } from "@maptiler/weather";
import { ref, shallowRef, onMounted, onUnmounted, markRaw } from 'vue';
import '@maptiler/sdk/dist/maptiler-sdk.css';

const mapContainer = shallowRef(null);
const map = shallowRef(null);

const cities = [
  { name: 'Toshkent', lat: 41.2995, lng: 69.2401 },
  { name: 'Samarqand', lat: 39.6542, lng: 66.9597 },
  { name: 'Buxoro', lat: 39.7747, lng: 64.4234 },
  { name: 'Andijon', lat: 40.7821, lng: 72.3442 },
  { name: 'Farg\'ona', lat: 40.3894, lng: 71.7837 },
  { name: 'Namangan', lat: 40.9983, lng: 71.6726 },
  { name: 'Nukus', lat: 42.4531, lng: 59.6103 },
  { name: 'Xiva', lat: 41.3783, lng: 60.3567 },
  { name: 'Qo\'qon', lat: 40.5286, lng: 70.9429 },
  { name: 'Navoiy', lat: 40.1039, lng: 65.3682 },
  { name: 'Termiz', lat: 37.2244, lng: 67.2785 },
  { name: 'Qoraqalpag\'iston', lat: 42.5787, lng: 59.2137 },
  { name: 'Jizzax', lat: 40.1286, lng: 67.8273 },
  { name: 'Qarshi', lat: 38.8634, lng: 65.7939 },
  { name: 'Shahrisabz', lat: 39.0588, lng: 66.8312 },
  { name: 'Xatirchi tumani', lat: 40.168676, lng: 66.019185 }, 
  {name: "Mo'ynoq", lat: 43.766978, lng: 59.027780},
  {name: "Uchquduq", lat: 42.150993, lng: 63.558182},
  {name: "Zarafshon", lat: 41.573827, lng: 64.189509}
];




 

const markerData = ref([]);

const getAQIColor = (aqi) => {
  if (aqi <= 50) return "#00E400";  // Good
  if (aqi <= 100) return "#FFFF00"; // Moderate
  if (aqi <= 150) return "#FF7E00"; // Unhealthy for sensitive groups
  if (aqi <= 200) return "#FF0000"; // Unhealthy
  if (aqi <= 300) return "#8F3F97"; // Very Unhealthy
  return "#7E0023";                 // Hazardous
}

async function fetchCityAQIData(city) {
  const url = `https://api.waqi.info/feed/geo:${city.lat};${city.lng}/?token=ba36341bc942385ecd64bb916f42288e67c586b5`;
  try {
    const response = await fetch(url);
    const data = await response.json();
    if (data.status === "ok") {
      const forecast = data.data.forecast;
      const iaqi = data.data.iaqi;
      return {
        ...city, 
        aqi: data.data.aqi, 
        dominantPollutant: data.data.dominentpol, 
        temp: iaqi.t?.v || "N/A", 
        humidity: iaqi.h?.v || "N/A", 
        windSpeed: iaqi.w?.v || "N/A", 
        forecast 
       };
    } else {
      console.error(`Error fetching AQI data for ${city.name}`);
    }
  } catch (error) {
    console.error(`Error fetching AQI data: ${error}`);
  }
  return null;
}

async function addAQIMarkers() {
  const promises = cities.map(fetchCityAQIData);
  const results = await Promise.all(promises);
  
  results.forEach((city) => {
    if (city && city.aqi !== undefined) {
      // Create a circular marker with AQI color
      const markerElement = document.createElement('div');
      markerElement.className = 'marker';
      markerElement.style.backgroundColor = getAQIColor(city.aqi);
      markerElement.style.borderRadius = '50%';
      markerElement.innerHTML = `<span>${city.aqi}</span>`;

      const marker = new Marker({ element: markerElement })
        .setLngLat([city.lng, city.lat])
        .addTo(map.value);

      // Add popup with AQI info
      const popup = new Popup({closeButton: false}).setHTML(`
      <h1 class='flex items-center gap-x-1 font-bold'>
        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24"><path fill="#919aa1" d="M12 11.5A2.5 2.5 0 0 1 9.5 9A2.5 2.5 0 0 1 12 6.5A2.5 2.5 0 0 1 14.5 9a2.5 2.5 0 0 1-2.5 2.5M12 2a7 7 0 0 0-7 7c0 5.25 7 13 7 13s7-7.75 7-13a7 7 0 0 0-7-7"/></svg>
        ${city.name}</h1>
        <p class='flex items-center gap-x-1'>Havo sifati: 
        ${city.aqi > 100 ? `<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 48 48"><g fill="#919aa1"><path d="M18 30h12v2H18zm12 4H18v2h12zm-8.698-11.558c.13-.358.091-.795-.016-1.193a4.2 4.2 0 0 0-.61-1.28c-.581-.829-1.544-1.59-2.845-1.646c-1.347-.056-2.353.799-2.973 1.706a5.6 5.6 0 0 0-.695 1.416c-.143.446-.219.902-.169 1.267a.5.5 0 0 0 .766.352c.4-.256.819-.607 1.207-.931c.176-.148.347-.29.505-.415c.562-.444 1-.697 1.363-.715c.344-.017.742.18 1.244.556c.18.134.353.276.534.424l.195.159c.244.197.504.399.766.557a.5.5 0 0 0 .728-.257m5.311 0c-.13-.358-.09-.795.017-1.193c.112-.416.319-.863.61-1.28c.58-.829 1.544-1.59 2.845-1.646c1.346-.056 2.353.799 2.973 1.706c.314.46.548.958.695 1.416c.142.446.218.902.168 1.267a.5.5 0 0 1-.765.352c-.4-.256-.82-.607-1.207-.931a25 25 0 0 0-.505-.415c-.563-.444-1-.697-1.363-.715c-.344-.017-.743.18-1.244.556c-.18.134-.354.276-.534.424l-.196.159a7 7 0 0 1-.765.557a.5.5 0 0 1-.729-.257"/><path fill-rule="evenodd" d="M41.853 26.315a17.95 17.95 0 0 1-5.336 10.62L36.5 37q-.28.243-.564.473A17.93 17.93 0 0 1 24 42a17.92 17.92 0 0 1-10.875-3.655q-.57-.4-1.125-.845l-.039-.118a17.97 17.97 0 0 1-5.792-10.904c-.902-.656-1.677-1.582-1.834-2.864c-.194-1.59.612-3.345 2.434-5.296a1 1 0 0 1 .203-.168C9.4 11.08 16.107 6 24 6c7.984 0 14.754 5.198 17.11 12.395c1.771 1.92 2.554 3.65 2.362 5.219c-.144 1.179-.812 2.057-1.62 2.701M14.308 36.732c6.38 4.468 14.025 4.323 20.332-.783l.081-.073c.41-1.665.6-2.88.567-4.037c-.03-1.042-.242-2.131-.755-3.531c-4.117-.863-7.348-1.296-10.54-1.308c-3.17-.011-6.378.393-10.443 1.279c-.449 1.487-.637 2.644-.633 3.752c.005 1.217.242 2.494.777 4.21q.3.252.614.491m-3.316-3.412c-.18-1.622-.036-3.135.451-4.948a19 19 0 0 1-.913-.166c-.625-.13-1.37-.318-2.12-.588a15.9 15.9 0 0 0 2.582 5.702m24.524-6.848c-8.94-1.919-14.151-2.01-23.116.013l-.27-.034a17 17 0 0 1-1.193-.204c-.911-.189-1.989-.492-2.887-.956A16 16 0 0 1 8 24c0-8.837 7.164-16 16-16s16 7.163 16 16q0 .597-.043 1.183l-.012.007c-.933.52-2.1.855-3.075 1.057a17 17 0 0 1-1.353.225m1.138 1.852c.547 1.693.716 3.113.605 4.632a15.9 15.9 0 0 0 2.347-5.416a15.6 15.6 0 0 1-2.33.666q-.333.069-.622.118" clip-rule="evenodd"/></g></svg>`: ''}
          ${city.aqi}</p>
        <p>Asosiy ifloslantiruvchi: ${city.dominantPollutant}</p>
        <p class='flex items-center gap-x-1'>Harorat:  <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24"><path fill="none" stroke="#919aa1" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 13.5a4 4 0 1 0 4 0V5a2 2 0 1 0-4 0zM4 9h4m5 7a4 4 0 1 0 0-8a4 4 0 0 0-1 .124M13 3v1m8 8h1m-9 8v1m6.4-15.4l-.7.7m0 11.4l.7.7"/></svg>
          ${city.temp} °C</p>
        <p class='flex items-center gap-x-1'>Namlik: <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24"><path fill="#919aa1" d="M12 21.5q-3.325 0-5.663-2.3T4 13.6q0-1.575.613-3.012T6.35 8.05l4.25-4.175q.3-.275.663-.425T12 3.3t.738.15t.662.425l4.25 4.175q1.125 1.1 1.738 2.538T20 13.6q0 3.3-2.337 5.6T12 21.5m-6-7.9h12q0-1.175-.45-2.237T16.25 9.5L12 5.3L7.75 9.5q-.85.8-1.3 1.863T6 13.6"/></svg> ${city.humidity}%</p>
        <p class='flex items-center gap-x-1'>Shamol tezligi: <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 256 256"><path fill="#919aa1" d="M184 184a32 32 0 0 1-32 32c-13.7 0-26.95-8.93-31.5-21.22a8 8 0 0 1 15-5.56C137.74 195.27 145 200 152 200a16 16 0 0 0 0-32H40a8 8 0 0 1 0-16h112a32 32 0 0 1 32 32m-64-80a32 32 0 0 0 0-64c-13.7 0-26.95 8.93-31.5 21.22a8 8 0 0 0 15 5.56C105.74 60.73 113 56 120 56a16 16 0 0 1 0 32H24a8 8 0 0 0 0 16Zm88-32c-13.7 0-26.95 8.93-31.5 21.22a8 8 0 0 0 15 5.56C193.74 92.73 201 88 208 88a16 16 0 0 1 0 32H32a8 8 0 0 0 0 16h176a32 32 0 0 0 0-64"/></svg> ${city.windSpeed} m/s</p>
       
      `);
      marker.setPopup(popup);
    }
  });
}

onMounted(async () => {
  config.apiKey = '3hORAuKJDvj8Oc7JyOw2'; // Replace with your API key

  const initialState = { lng: 69.24074, lat: 41.31086, zoom: 5 };

  map.value = markRaw(new Map({
    container: mapContainer.value,
    style: MapStyle.STREETS,
    center: [initialState.lng, initialState.lat],
    zoom: initialState.zoom,
  }));

  // Add weather layers
  
  // Temperature will be used as the main overlay
  const temperatureLayer = new TemperatureLayer({
          opacity: 0.7,
        });

  // Radar will be using the cloud color ramp and used as a cloud overlay
      const radarLayer = new RadarLayer({
          colorramp: ColorRamp.builtin.RADAR_CLOUD,
        });

  const windLayer = new WindLayer({
    colorramp: ColorRamp.builtin.NULL,
    color: [255, 255, 255, 0],
    fastColor: [255, 255, 255, 100],
  });


  const precipitationLayer = new PrecipitationLayer({
          colorramp: ColorRamp.builtin.NULL,
  });

  map.value.on('load', () => {
    map.value.setPaintProperty("Water", 'fill-color', "rgba(0, 0, 0, 0.7)");
    map.value.addLayer(temperatureLayer, "Place labels");
    map.value.addLayer(windLayer);
    map.value.addLayer(radarLayer);
    map.value.addLayer(precipitationLayer);
  });

  map.value.on("click", () => {
  console.log("clicked")
})
  // Add AQI markers
  await addAQIMarkers();
});





onUnmounted(() => {
  map.value?.remove();
});
</script>

<style>
.map-wrap {
  position: relative;
  width: 100%;
  height: 100vh;
}

.map {
  position: absolute;
  width: 100%;
  height: 100%;
}

.marker {
  width: 25px;
  height: 25px;
  color: rgb(0, 0, 0);
  display: flex;
  justify-content: center;
  align-items: center;
}

.legend {
  position: absolute;
  bottom: 20px;
  right: 20px;
  font-size: 14px;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.legend-color {
  display: inline-block;
  width: 20px;
  height: 20px;
  border-radius: 5px;
  margin-right: 10px;
}
</style>
