<script>
  import { tick } from "svelte";
  import L from "leaflet";
  import "leaflet/dist/leaflet.css";
  import { getDistance } from "geolib";

  let position = null;
  let error = null;
  let address = null;
  let map;
  let distanceToTarget = null;

  const target = { latitude: 41.328403, longitude: 19.818312 };

  async function showPosition(pos) {
    position = pos;
    error = null;

    const latitude = pos.coords.latitude;
    const longitude = pos.coords.longitude;

    distanceToTarget = getDistance(
      { latitude, longitude },
      target
    );

    await tick();

    if (!map) {
      map = L.map("map").setView([latitude, longitude], 13);

      L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
        maxZoom: 19,
        attribution:
          '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
      }).addTo(map);
    } else {
      map.setView([latitude, longitude], 13);
    }

    L.marker([latitude, longitude])
      .addTo(map)
      .bindPopup("You are here")
      .openPopup();

    fetch(
      `https://api.geoapify.com/v1/geocode/reverse?lat=${latitude}&lon=${longitude}&apiKey=466b5735f3a04b5cba388e6295d6d51b`
    )
      .then((response) => response.json())
      .then((result) => {
        address =
          result.features?.[0]?.properties?.formatted || "No address found";
      })
      .catch((err) => {
        error = "Error: " + err.message;
      });
  }

  function showError(err) {
    error = err.message;
    position = null;
  }

  function getCurrentPosition() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(showPosition, showError);
    } else {
      error = "Geolocation is not supported by this browser.";
    }
  }

  function formatKm(meters) {
    return (meters / 1000).toFixed(2);
  }
</script>

<div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-100 via-white to-blue-200">
  <div class="bg-white rounded-2xl shadow-xl p-8 max-w-md w-full text-center space-y-6">
    <h1 class="text-3xl font-bold text-gray-800">Geolocation Finder</h1>
    <p class="text-gray-600">Click below to detect your current location</p>

    <button
      on:click={getCurrentPosition}
      class="px-6 py-3 rounded-xl bg-blue-500 text-white font-semibold shadow-md hover:bg-blue-600 transition"
    >
      Get Location & Distance
    </button>

    {#if position}
      <div class="mt-6 bg-blue-50 p-4 rounded-xl text-left shadow-inner space-y-2">
        <p><span class="font-semibold">Latitude:</span> {position.coords.latitude}</p>
        <p><span class="font-semibold">Longitude:</span> {position.coords.longitude}</p>
        <p><span class="font-semibold">Accuracy:</span> {position.coords.accuracy} meters</p>

        {#if distanceToTarget !== null}
          <p><span class="font-semibold">Distance to target:</span> 
            {distanceToTarget} m ({formatKm(distanceToTarget)} km)
          </p>
        {/if}

        {#if address}
          <p class="text-green-700 font-medium">{address}</p>
        {/if}

        <div id="map" class="h-64 w-full rounded-xl shadow-md"></div>
      </div>
    {:else if error}
      <div class="mt-6 bg-red-50 p-4 rounded-xl text-left shadow-inner">
        <p class="text-red-600 font-semibold">{error}</p>
      </div>
    {/if}
  </div>
</div>