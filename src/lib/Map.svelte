<script lang="ts">
    import { onMount } from 'svelte';
    import L from 'leaflet';

    // Get your coordinates from props
    let { lat, lon } = $props();
    let mapElement: HTMLElement;
    let map: L.Map;

    onMount(() => {
        // NO 'let' here! Use the variable we declared above
        map = L.map(mapElement).setView([lat, lon], 15);
        
        L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
            attribution: '&copy; OpenStreetMap'
        }).addTo(map);

        navigator.geolocation.getCurrentPosition((position) => {
            const myLat = position.coords.latitude;
            const myLon = position.coords.longitude;

            map.flyTo([myLat, myLon], 18)

            L.marker([myLat, myLon]).addTo(map).bindPopup("You are here")
        })
    });

    export function getCurrentLocation() {
    	navigator.geolocation.getCurrentPosition((position) => {
			const { latitude, longitude } = position.coords;

			// Fly the map to the actual feet
			map.flyTo([latitude, longitude], 18)

			L.marker([latitude, longitude])
			.addTo(map)
			.bindPopup("You are here")
		})
    }
</script>

<div bind:this={mapElement} id="map"></div>

<style>
    #map {
        height: 400px;
        width: 100%;
        border-radius: 8px;
    }
</style>