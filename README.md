**Javascript** | current release: **1.0.3** build 2022-04-30

# Map Markers Geocode

Javascript code examples showing how to use markers and geocoding on map APIs using two different engines: **Google Maps** and **Open Street Map** (using the Leaflet library - https://leafletjs.com). The examples contain all the necessary libraries and are commented out. The sample code can be used as a tutorial and training material when implementing your own solutions based on Google Maps or Open Street Map.

**The repository contains working code examples in 2 files:**

`google_maps.html` - shows map marker placement and coordinate search with geocoding using Google Maps

`open_street_map.html` - shows map marker placement and coordinate search with geocoding using Open Street Map

Remember to use your API key in the Google Maps example, which should be entered at the bottom of the page in place of *{YOUR_API_KEY}*:

```js
<script src = "https://maps.googleapis.com/maps/api/js?key={YOUR_API_KEY}&callback=initMap&v=weekly" async></script>
```

# Online example:
You can also see the live examples in action in the online demo at:
### https://szczyglis.dev/map-markers-geocode

**INFO: The Google Maps online demo uses a developer API key with geocoding turned off.**

![map1](https://user-images.githubusercontent.com/61396542/166083596-77a142e9-e408-420c-9e37-1a5aa0edf6ac.png)

![map2](https://user-images.githubusercontent.com/61396542/166083603-cc374586-297b-4149-83e8-5c452da9c66a.png)

# Tutorial: how to use maps API's?

## Google Maps / Javascript
**The example shows how to place the draggable marker on the map, read the coordinates from its position, and how to use geocoding to find the coordinates of the place you are looking for, and then update the marker position with the coordinates found.**

For the full working example, including the form, libraries, and browser-working code, see the file `google_maps.html` in the repository.

```js
let map,
    marker,
    geocoder;

// Initializes google map
function initMap() {

    console.log('Initializing map..');

    // Initialize google geocoder
    geocoder = new google.maps.Geocoder();

    // Define map initial parameters
    const myLatlng = new google.maps.LatLng(40.7127753,-74.0059728); // New York				    
    const mapOptions = {
      zoom: 6,
      center: myLatlng,
      mapTypeId: 'hybrid'
    }

    // Create map in "map" div
    map = new google.maps.Map(document.getElementById('map'), mapOptions);

    // Create marker
    marker = new google.maps.Marker({
        position: myLatlng,
        map: map,
        draggable:true,
        title:"Drag marker"
    });

    // Add dragend listener to marker
    marker.addListener("dragend", () => {
        const pos = marker.getPosition();
        const lat = pos.lat();
        const lng = pos.lng();
        $('input[name="lat"]').val(lat);
        $('input[name="lng"]').val(lng);
        console.log('On marker drag end', lat, lng);
      });

    // Add click listener to marker
    google.maps.event.addListener(map, 'click', function(event) {
        marker.setPosition(event.latLng);
        $('input[name="lat"]').val(event.latLng.lat());
        $('input[name="lng"]').val(event.latLng.lng());
        console.log('On map click', event.latLng.lat(), event.latLng.lng());
    });
}

// Executes geocode search
function geocode(request) {
  geocoder
    .geocode(request)
    .then((result) => {
      const { results } = result;
      map.setCenter(results[0].geometry.location);
      map.setZoom(10);
      marker.setPosition(results[0].geometry.location);
      $('input[name="lat"]').val(results[0].geometry.location.lat);
      $('input[name="lng"]').val(results[0].geometry.location.lng);
      console.log('Geocode result', results);
      return results;
    })
    .catch((e) => {
      alert("Geocode error: " + e);
    });
};

// Updates marker position on map
function updateMarker() {
    const lat = $('input[name="lat"]').val().trim();
    const lng = $('input[name="lng"]').val().trim();
    if (lat.length != 0 && lng.length != 0) {
        const latlng = new google.maps.LatLng(lat, lng);
        marker.setPosition(latlng);
        //map.setCenter(latlng); // < --- uncomment if you want to center map in this place
        console.log('Update marker position', lat, lng);
    }
};

$(document).ready(function() {
    // Refresh marker position when manual latitude update
    $('body').on('change', 'input[name="lat"]', function(e) {
        updateMarker();
    });

    // Refresh marker position when manual longitude update
    $('body').on('change', 'input[name="lng"]', function(e) {
        updateMarker();
    });

    // Run geocode search on form submit
    $('body').on('submit', 'form[name="form"]', function (e) {
        e.preventDefault();
        const query = $('input[name="search_query"]').val();
        if (query != '') {
            geocode({ address: query });
        }
    });

    // Run geocode search on click search button
    $('body').on('click', 'button[id="search"]', function(e) {
      e.preventDefault();
        const query = $('input[name="search_query"]').val();
        if (query != '') {
            geocode({ address: query });
        }
    });
});
```

## Open Street Map / Javascript
**The example shows how to place the draggable marker on the map, read the coordinates from its position, and how to use geocoding to find the coordinates of the place you are looking for, and then update the marker position with the coordinates found.**

For the full working example, including the form, libraries, and browser-working code, see the file `open_street_map.html` in the repository.

```js
let map, 
    marker;

function geocode(query) {
    $.ajax({
        url: 'https://nominatim.openstreetmap.org/search/'+query+'?format=json',
        dataType: 'json',
        method: 'GET'
    })
      .done(function (data) {
          if (typeof data[0]['lat'] !== 'undefined' && typeof data[0]['lon'] !== 'undefined') {
            const lat = data[0]['lat'];
            const lng = data[0]['lon'];
            $('input[name="lat"]').val(lat);
            $('input[name="lng"]').val(lng);
            setPoint(lat, lng);
            console.log('Geocode result', lat, lng);
          }
      })
      .fail(function (err) {
          console.error(err);
      });
};

// Click on map
function onMapClick(e) {
    const lat = e.latlng.lat;
    const lng = e.latlng.lng;
    marker.setLatLng(e.latlng);
    marker.setPopupContent(lat + "," + lng).openPopup();
    $('input[name="lat"]').val(lat);
    $('input[name="lng"]').val(lng);
    console.log('On map click', lat, lng);
}

// Marker drag end
function onDragEnd(e) {
    const lat = e.target._latlng.lat;
    const lng = e.target._latlng.lng;
    marker.setLatLng(e.target._latlng);
    $('input[name="lat"]').val(lat);
    $('input[name="lng"]').val(lng);
    console.log('On marker drag end', lat, lng);
}

// Sets marker position
function setPoint(lat, lng) {
    marker.setLatLng(L.latLng(lat, lng));
    map.setView([lat, lng], 12);
}

// Updates marker position on map
function updateMarker() {
    const lat = $('input[name="lat"]').val().trim();
    const lng = $('input[name="lng"]').val().trim();
    if (lat.length != 0 && lng.length != 0) {
      setPoint(lat, lng);
      // map.setView([lat, lng], 12); // < --- uncomment if you want to center map in this place
        console.log('Update marker position', lat, lng);
    }
};

map = L.map('map');
marker = L.marker([40.7127753, -74.0059728], {
  draggable:'false'
});
marker.on('dragend', onDragEnd);
marker.bindPopup('0,0');
marker.addTo(map);
marker.setPopupContent('40.7127753,-74.0059728');

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png?{foo}', { 
    foo: 'bar',
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>' 
}).addTo(map),
setPoint(40.7127753, -74.0059728);
map.setZoom(6);
map.on('click', onMapClick);

$(document).ready(function() {
    // Refresh marker position when manual latitude update
    $('body').on('change', 'input[name="lat"]', function(e) {
        updateMarker();
    });

    // Refresh marker position when manual longitude update
    $('body').on('change', 'input[name="lng"]', function(e) {
        updateMarker();
    });

    // Run geocode search on form submit
    $('body').on('submit', 'form[name="form"]', function (e) {
        e.preventDefault();
        const query = $('input[name="search_query"]').val();
        if (query != '') {
            geocode(query);
        }
    });

    // Run geocode search on click search button
    $('body').on('click', 'button[id="search"]', function(e) {
      e.preventDefault();
        const query = $('input[name="search_query"]').val();
        if (query != '') {
            geocode(query);
        }
    });
});
```
___

# Changelog
**- 1.0.3** - Published first release. (2022-04-30)

# Credits
 
### Published examples are free to use but if you liked then you can donate this work via BTC: 

**14X6zSCbkU5wojcXZMgT9a4EnJNcieTrcr**

or by PayPal:
 **[https://www.paypal.me/szczyglinski](https://www.paypal.me/szczyglinski)**


**Enjoy!**

MIT License | 2022 Marcin 'szczyglis' Szczygliński

https://github.com/szczyglis-dev/extended-dump-bundle

https://szczyglis.dev

Contact: szczyglis@protonmail.com
