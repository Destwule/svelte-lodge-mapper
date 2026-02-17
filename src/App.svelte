<script lang="ts">
  import Map from './lib/Map.svelte';

  let mapComponent; // We use this to call the recordLodge function
  const lafia_latitude = 8.479;
  const lafia_longitude = 8.527;
</script>

<main>
  <h1>Lodge Mapper</h1>

  <div class="controls">
    <button onclick={() => mapComponent.recordCurrentLocation()}>
      Record Current Lodge
    </button>
  </div>

  <Map bind:this={mapComponent} lat={lafia_latitude} lon={lafia_longitude} />
</main>

<style>
  main {
    padding: 20px;
    font-family: sans-serif;
  }
  .controls {
    margin-bottom: 10px;
  }
  button {
    padding: 10px;
    background: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
</style>