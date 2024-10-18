<script>
    import { onMount, onDestroy } from 'svelte';
    import 'leaflet/dist/leaflet.css';

    export let inputValue = 1; // Přijímá hodnotu z +page.svelte
    export let selectedFeatureTypes = []; // Přijímá typy z +page.svelte
     // Reakce na změnu vybraných typů

    let L; 
    let map; 
    let features = [];
    let userLocation = { lat: null, lon: null };
    const defaultRadius = 100; // Výchozí radius
    const apiKey = 'f9565ebb-f8e3-4b6c-bdce-d91d42a8ec9b'; // API klíč
    let transportMode = 'foot'; // Výchozí způsob dopravy

    const sources = [
        {
            url: 'https://services6.arcgis.com/ogJAiK65nXL1mXAW/arcgis/rest/services/Praktičtí_lékaři_pro_dospělé/FeatureServer/0/query?outFields=*&where=1%3D1&f=geojson',
            icon: 'src/Icons/person-solid.svg',
            type: 'prakticky'
        },
        {
            url: 'https://services6.arcgis.com/ogJAiK65nXL1mXAW/arcgis/rest/services/Zubní_lékaři/FeatureServer/0/query?outFields=*&where=1%3D1&f=geojson',
            icon: 'src/Icons/tooth-solid.svg',
            type: 'zubni'
        },
        {
            url: 'https://services6.arcgis.com/ogJAiK65nXL1mXAW/arcgis/rest/services/Ambulantní_gynekologové/FeatureServer/0/query?outFields=*&where=1%3D1&f=geojson',
            icon: 'src/Icons/venus-mars-solid.svg',
            type: 'gynekolog'
        },
        {
            url: 'https://services6.arcgis.com/ogJAiK65nXL1mXAW/arcgis/rest/services/Praktičtí_lékaři_pro_děti_a_dorost/FeatureServer/0/query?outFields=*&where=1%3D1&f=geojson',
            icon: 'src/Icons/baby-solid.svg',
            type: 'detsky'
        }
    ];

    onMount(async () => {
        L = (await import('leaflet')).default;

        const dataPromises = sources.map(source => fetch(source.url).then(response => response.json()));
        const allData = await Promise.all(dataPromises);

        features = allData.flatMap((data, index) => {
            return data.features.map(feature => ({
                ...feature,
                icon: sources[index].icon,
                type: sources[index].type
            }));
        });

        // Inicializace mapy
        map = L.map('map').setView([50.0755, 14.4378], 8);

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '© OpenStreetMap'
        }).addTo(map);

        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(position => {
                userLocation.lat = position.coords.latitude;
                userLocation.lon = position.coords.longitude;

                updateCircleAndMarkers();
                map.setView([userLocation.lat, userLocation.lon], 13);
            }, () => {
                console.error('Nelze získat polohu.');
            });
        } else {
            console.error('Geolokace není podporována tímto prohlížečem.');
        }
    });

    function updateCircleAndMarkers() {
        const radius = (inputValue > 0) ? inputValue * 100 : defaultRadius;

        if (map.circle) {
            map.removeLayer(map.circle);
        }
        map.circle = L.circle([userLocation.lat, userLocation.lon], {
            color: 'blue',
            fillColor: '#30f',
            fillOpacity: 0.5,
            radius: radius
        }).addTo(map);

        showMarkersInRadius(radius);
    }

    function showMarkersInRadius(radius) {
        map.eachLayer(layer => {
            if (layer instanceof L.Marker) {
                map.removeLayer(layer);
            }
        });

        features.forEach(feature => {
            const coords = feature.geometry.coordinates;
            const markerLat = coords[1];
            const markerLon = coords[0];
            const distance = map.distance([userLocation.lat, userLocation.lon], [markerLat, markerLon]);

            if (distance <= radius) {
                // Filtrovat podle selectedFeatureTypes
                if (selectedFeatureTypes.length > 0 && !selectedFeatureTypes.includes(feature.type)) {
                    return; // Pokud neodpovídá, přeskoč
                }

                const marker = L.marker([markerLat, markerLon], {
                    icon: L.icon({
                        iconUrl: feature.icon,
                        iconSize: [25, 25]
                    })
                }).addTo(map);
                marker.bindPopup(feature.properties.provozovatel);
                
                marker.on('click', async () => {
                    const distance = await getDistanceFromGraphHopper(userLocation, { lat: markerLat, lng: markerLon });  
                    const transportText = transportMode === 'car' ? 'vzdálenost autem' : 'vzdálenost pěšky';
                    marker.bindPopup(`${feature.properties.provozovatel}<br>${transportText}: ${(distance / 1000).toFixed(2)} km`).openPopup();
                });
            }
        });
    }

    async function getDistanceFromGraphHopper(start, end) {
        const response = await fetch(`https://graphhopper.com/api/1/route?point=${start.lat},${start.lon}&point=${end.lat},${end.lng}&vehicle=${transportMode}&key=${apiKey}`);
        
        if (!response.ok) {
            console.error('API Error:', response.status, await response.text());
            throw new Error('Nelze získat trasu.');
        }

        const data = await response.json();
        if (data.paths && data.paths.length > 0) {
            return data.paths[0].distance; // Vzdálenost v metrech
        } else {
            throw new Error('Nelze získat trasu.');
        }
    }

    function changeTransportMode(mode) {
        transportMode = mode;
        updateCircleAndMarkers(); // Aktualizace markerů podle nového způsobu dopravy
    }

    $: inputValue, userLocation.lat && userLocation.lon ? updateCircleAndMarkers() : null;
    $: selectedFeatureTypes, userLocation.lat && userLocation.lon ? updateCircleAndMarkers() : null; // Reakce na změnu vybraných typů
    onDestroy(() => {
        if (map) {
            map.remove();
        }
    });
</script>

<style>
    #map {
        width: 100%;
        height: 500px;
    }

    /* New styles for the map container */
    .map-container {
        max-width: 800px; /* Set a maximum width */
        margin: 0 auto; /* Center the map */
        padding: 20px; /* Space around the map */
        border: 2px solid #ccc; /* Border around the map */
        border-radius: 12px; /* Rounded corners */
        background-color: #f9f9f9; /* Light background color */
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Subtle shadow */
    }

    /* Styl pro tlačítka dopravy */
    .transport-mode button {
        display: block;
        width: 100px; /* Šířka tlačítek */
        padding: 15px; /* Výška tlačítek */
        margin: 10px 0; /* Vzdálenost mezi tlačíky */
        border: none;
        background-color: #007BFF; /* Primární barva tlačítek */
        color: white; /* Barva textu */
        border-radius: 12px; /* Zaoblené rohy */
        font-size: 16px; /* Velikost textu */
        font-weight: bold; /* Tučný text */
        cursor: pointer;
        transition: background-color 0.3s, transform 0.2s; /* Přechod při najetí a kliknutí */
    }

    .transport-mode button:hover {
        background-color: #0056b3; /* Tmavší barva při najetí myší */
    }

    .transport-mode button:active {
        transform: scale(0.95); /* Mírné zmenšení při kliknutí */
    }

    .transport-mode button.active {
        background-color: #28a745; /* Změna barvy při aktivním stavu */
        box-shadow: 0 8px 16px rgba(0, 123, 255, 0.2);
    }
</style>

<div class="gap-5 transport-mode flex items-center justify-center">
    <button class:active={transportMode === 'foot'} on:click={() => changeTransportMode('foot')}>Pěšky</button>
    <button class:active={transportMode === 'car'} on:click={() => changeTransportMode('car')}>Autem</button>
</div>

<!-- New map container -->
<div class="map-container">
    <div id="map"></div>
</div>

