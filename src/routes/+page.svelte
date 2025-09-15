<script>
    let position = null;
    let error = null;

    function showPosition(pos) {
        position = pos;
        error = null;
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
</script>

<div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-100 via-white to-blue-200">
  <div class="bg-white rounded-2xl shadow-xl p-8 max-w-md w-full text-center space-y-6">
    <h1 class="text-3xl font-bold text-gray-800">🌍 Geolocation Finder</h1>
    <p class="text-gray-600">Click below to detect your current location</p>

    <button 
      on:click={getCurrentPosition}
      class="px-6 py-3 rounded-xl bg-blue-500 text-white font-semibold shadow-md hover:bg-blue-600 transition">
      Get Location
    </button>

    {#if position}
      <div class="mt-6 bg-blue-50 p-4 rounded-xl text-left shadow-inner space-y-2">
        <p class="text-gray-800"><span class="font-semibold">Latitude:</span> {position.coords.latitude}</p>
        <p class="text-gray-800"><span class="font-semibold">Longitude:</span> {position.coords.longitude}</p>
        <p class="text-gray-800"><span class="font-semibold">Accuracy:</span> {position.coords.accuracy} meters</p>
      </div>
    {:else if error}
      <div class="mt-6 bg-red-50 p-4 rounded-xl text-left shadow-inner">
        <p class="text-red-600 font-semibold">⚠️ {error}</p>
      </div>
    {/if}
  </div>
</div>