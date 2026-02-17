<script lang="ts">
    import { onMount } from 'svelte';
    import L from 'leaflet';

    type Location = {
        id: string,
        location_name: string,
        lat: number;
        lon: number;
    }

    let mapElement: HTMLDivElement;
    let map: L.Map;
    let popupBlueprint: HTMLElement;
    let locations = $state([]);
    let { lat, lon } = $props();

    const makeLocationObject = (lat: number, lon: number, location_name: string): Location => {
        const newLocation = {
            id: crypto.randomUUID(),
            location_name: location_name,
            lat: lat,
            lon: lon,
        }

        return newLocation
    }

    const addMarkerPopup = (location_obj: Location) => {
        const marker = L.marker([location_obj.lat, location_obj.lon]).addTo(map);
        const content = popupBlueprint.cloneNode(true) as HTMLElement;

        content.querySelector("#location-name")!.textContent = location_obj.location_name
        marker.bindPopup(content)
    }


    onMount(() => {
        map = L.map(mapElement).setView([lat, lon], 18);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);
    })


    export const recordCurrentLocation = () => {
        console.log("About to show the current location")
        navigator.geolocation.getCurrentPosition((position) => {
            const lat = position.coords.latitude;
            const lon = position.coords.longitude;

            map.flyTo([lat, lon], 18);
            const newMarker = L.marker([lat, lon], {
                draggable: true,
            }).addTo(map)

            let location_name = "You are here";
            const newLocation = makeLocationObject(lat, lon, location_name)
            addMarkerPopup(newLocation)
        })
    }

</script>


<div style="display: none;">
    <div bind:this={popupBlueprint} class="lodge-popup">
        <b class="location-name">Location Name</b>
        <hr>
        <div style="display: flex; gap: 10px;">
            <button class="editBtn">Edit</button>
            <button class="deleteBtn">Delete</button>
        </div>
    </div>
</div>

<div bind:this={mapElement} id="map"></div>

<style>
    #map {
        height: 400px;
        width: 100%;
        border-radius: 8px;
    }
</style>