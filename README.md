# yahnjr.github.io
Testing web apps
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>La Pine Map</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://api.mapbox.com/mapbox-gl-js/v3.2.0/mapbox-gl.css" rel="stylesheet">
    <script src="https://api.mapbox.com/mapbox-gl-js/v3.2.0/mapbox-gl.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
        }
        #map {
            position: absolute;
            top: 0;
            bottom: 0;
            width: 100%;
        }
        .type-buttons {
            position: absolute;
            top: 10px;
            right: 10px;
            z-index: 1;
        }
        .type-button {
            width: 40px;
            height: 40px;
            background-color: #f0f0f0;
            border: none;
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: background-color 0.3s ease; /* Add transition effect */
        }
        .type-button:hover {
            background-color: #ddd;
        }
        .selected {
            background-color: #333; /* Darken the selected button */
            color: #fff; /* Set text color to white */
        }
        /* Tooltip container */
        .tooltip {
            position: relative;
            display: inline-block;
        }
        /* Tooltip text */
        .tooltip .tooltiptext {
            visibility: hidden;
            width: 120px;
            background-color: black;
            color: #fff;
            text-align: center;
            border-radius: 6px;
            padding: 5px 0;
            /* Position the tooltip */
            position: absolute;
            z-index: 1;
            bottom: 100%;
            left: 50%;
            margin-left: -60px;
        }
        /* Show the tooltip text when you mouse over the tooltip container */
        .tooltip:hover .tooltiptext {
            visibility: visible;
        }
    </style>
</head>
<body>
<div id="map"></div>
<div class="type-buttons">
    <div class="tooltip"><button class="type-button" onclick="setType('Housing')">🏡<span class="tooltiptext">Housing</span></button></div>
    <div class="tooltip"><button class="type-button" onclick="setType('Infrastructure')">🏭<span class="tooltiptext">Infrastructure</span></button></div>
    <div class="tooltip"><button class="type-button" onclick="setType('Transportation')">🚎<span class="tooltiptext">Transportation</span></button></div>
    <div class="tooltip"><button class="type-button" onclick="setType('Employment')">💼<span class="tooltiptext">Employment</span></button></div>
    <div class="tooltip"><button class="type-button" onclick="setType('Parks and Nature')">🌳<span class="tooltiptext">Parks & Nature</span></button></div>
</div>
<script>
    mapboxgl.accessToken = 'pk.eyJ1IjoiaWFubWFoZXIzaiIsImEiOiJjbHRnM2g3Mmgwdm50MmpxcjNiaHppcGF0In0.EDCKHSTyRqogqjRVwC5pJA';

    const map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/ianmaher3j/cltxk9cdn01ij01r53gz169jm',
        center: [-121.50527954101562, 43.66847610473633],
        zoom: 12,
        doubleClickZoom: false // Disable zoom on double click
    });
    
    map.on('load', () => {
        map.addSource('lapine-comments', {
            type: 'geojson',
            data: 'https://services3.arcgis.com/pZZTDhBBLO3B9dnl/arcgis/rest/services/LaPineComments/FeatureServer/0/query?where=1%3D1&outFields=*&outSR=4326&f=geojson'
        });

        map.addLayer({
            id: 'lapine-comments-layer',
            type: 'circle',
            source: 'lapine-comments',
            paint: {
                // Use a match expression to dynamically set circle color based on the 'type' property
                'circle-color': [
                    'match',
                    ['get', 'Type'],
                    'Housing', '#FF5733',
                    'Infrastructure', '#3498DB',
                    'Transportation', '#E3F710',
                    'Employment', '#800080',
                    'Parks and Nature', '#00FF00',
                    /* other */ '#000'
                ],
                'circle-radius': 6,
                'circle-stroke-width': 2,
                'circle-stroke-color': 'white'
            }
        });

        // Add a click event listener for displaying popups
        map.on('click', 'lapine-comments-layer', (e) => {
            const coordinates = e.features[0].geometry.coordinates.slice();
            const description = `<strong>${e.features[0].properties.Type}</strong><br>${e.features[0].properties.Comment}`;

            // Ensure that if the map is zoomed out such that multiple
            // copies of the feature are visible, the popup appears
            // over the copy being pointed to.
            while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
            }

            new mapboxgl.Popup()
                .setLngLat(coordinates)
                .setHTML(description)
                .addTo(map);
        });
    });
  
    let selectedType = 'Housing'; // Default type

    function setType(type) {
        selectedType = type;

        // Remove 'selected' class from all buttons
        document.querySelectorAll('.type-button').forEach(button => {
            button.classList.remove('selected');
        });

        // Add 'selected' class to the clicked button
        document.querySelector(`[onclick="setType('${type}')"]`).classList.add('selected');
    }

    map.on('dblclick', (e) => {
        const comment = prompt('Add a short comment (maximum 250 characters):');
        if (comment === null || comment.trim() === '') {
            return;
        }

        // Create a marker at the clicked location with the selected type
        const marker = new mapboxgl.Marker({
            color: getMarkerColor(selectedType),
        })
            .setLngLat(e.lngLat)
            .setPopup(new mapboxgl.Popup().setHTML(`<strong>${selectedType}</strong><br>${comment}`))
            .addTo(map);
    });

    function getMarkerColor(type) {
        switch (type) {
            case 'Housing':
                return '#FF5733'; // Orange
            case 'Infrastructure':
                return '#3498DB'; // Blue
            case 'Transportation':
                return '#E3F710'; // Orange
            case 'Employment':
                return '#800080'; // Purple
            case 'Parks and Nature':
                return '#00FF00'; // Bright Green
            default:
                return '#000'; // Default black
        }
    }
</script>
</body>
</html>






