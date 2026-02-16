<script lang="ts">
    import { onMount } from 'svelte';
    import L from 'leaflet';

    type Location = {
        id: string,
        location_name: string,
        lat: number,
        lon: number
    }
    type location_params = (
        latitude: number,
        longitude: number,
        location_name: string
    ) => Location;

    // Get your coordinates from props
    const LOCATIONS_KEY = "LOCATIONS"
    let { lat, lon } = $props();
    let mapElement: HTMLElement;
    let map: L.Map;
    let locations: Location[];

    $effect(() => {
        const savedData = localStorage.getItem(LOCATIONS_KEY)
        locations = savedData ? JSON.parse(savedData) : [];
    })

    const make_location_object: location_params = (latitude, longitude, location_name) => {
        const newLocation: Location = {
            id: crypto.randomUUID(),
            location_name: location_name,
            lat: latitude,
            lon: longitude
        }

        return newLocation
    }

    const save_location = (location_obj: Location) => {
        const newLocation = location_obj
        locations.push(newLocation);

        save_location_list()
    }

    const save_location_list = (): void => {
        localStorage.setItem(LOCATIONS_KEY, JSON.stringify(locations));
    }

    const remove_location = (ID: string) => {
        locations = locations.filter((location) => location.id !== ID)

        save_location_list()
    }


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

            const marker = L.marker([myLat, myLon]).addTo(map)
            const markerText = "Current Location"

            addMarkerPopup(marker, markerText)
            
            const newLocation = make_location_object(myLat, myLon, markerText)
            save_location(newLocation)
        })
    });


    const addMarkerPopup = (newMarker: L.Marker, location_name: string) => {
        // 1. Store the name in a local variable so we can change it
        let currentName = location_name;

        // 2. Create a function that builds the popup content
        const renderPopup = () => {
            const container = document.createElement('div');
            container.innerHTML = `
                <div style="text-align: center;">
                    <b id="name-display" class="nameDisplay">${currentName}</b>
                    <div style="display: flex; gap: 5px; justify-content: center;">
                        <button id="edit-btn" class="edit-btn">Edit</button>
                        <button id="delete-btn" class="deleteBtn">Delete</button>
                    </div>
                </div>
            `;

            // EDIT LOGIC
            container.querySelector("#edit-btn").onclick = () => {
                const newName = prompt("Edit Lodge Name:", currentName);
                if (newName !== null && newName.trim() !== "") {
                    currentName = newName;
                    renderPopup(); // Re-run this function to update the UI
                }
            };

            // DELETE LOGIC
            container.querySelector("#delete-btn").onclick = () => {
                if (confirm("Delete this marker?")) {
                    newMarker.remove();
                }
            };

            // Update the marker with the new container
            newMarker.bindPopup(container).openPopup();
        };  

        // Initial run to set up the popup
        renderPopup();
    }


    export function recordLodge() {
        const lodgeName = prompt("What is the name of the lodge") || "Nameless Lodge"
    	navigator.geolocation.getCurrentPosition((position) => {
			const { latitude, longitude } = position.coords;

			// Fly the map to the actual feet
			map.flyTo([latitude, longitude], 18)

			const marker = L.marker([latitude, longitude], {
                draggable: true,
            }).addTo(map)
            
            addMarkerPopup(marker, lodgeName)
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