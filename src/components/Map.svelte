<script>
  import { onMount, onDestroy } from 'svelte'
  import maplibregl, { NavigationControl, Marker} from 'maplibre-gl';
  import 'maplibre-gl/dist/maplibre-gl.css';



  onMount(() => {

    // const coordinates = document.getElementById('coordinates');
    // const apiKey = 'VWgJ4y0zBULXVnp2tqkp ';

    const initialState = { lng: 123.8854, lat: 10.3157 , zoom: 10 };

    var map = new maplibregl.Map({
      container: 'map',
      // style: `https://api.maptiler.com/maps/streets/style.json?key=${apiKey}`,
      style: {
            'version': 8,
            'name': 'Blank',
            'center': [0, 0],
            'zoom': 0,
            'sources': {
                'raster-tiles': {
                    'type': 'raster',
                    'tiles': ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
                    'tileSize': 256,
                    'minzoom': 5,
                    'maxzoom': 30
                }
            },
            'layers': [
                {
                    'id': 'background',
                    'type': 'background',
                    'paint': {
                        'background-color': '#e0dfdf'
                    }
                },
                {
                    'id': 'simple-tiles',
                    'type': 'raster',
                    'source': 'raster-tiles'
                }
            ],
            'id': 'blank'
        },
      // center: [-87.61694, 41.86625],
      // zoom: 15.99,
      center: [initialState.lng, initialState.lat],
      zoom: initialState.zoom,
      pitch:0,
      bearing:0,
      antialias:true
    });

    var geocoder_api = {
        forwardGeocode: async (config) => {
            const features = [];
            try {
                let request =
                    'https://nominatim.openstreetmap.org/search?q=' +
                    config.query +
                    '&format=geojson&polygon_geojson=1&addressdetails=1';
                const response = await fetch(request);
                const geojson = await response.json();
                for (let feature of geojson.features) {
                    let center = [
                        feature.bbox[0] +
                            (feature.bbox[2] - feature.bbox[0]) / 2,
                        feature.bbox[1] +
                            (feature.bbox[3] - feature.bbox[1]) / 2
                    ];
                    let point = {
                        type: 'Feature',
                        geometry: {
                            type: 'Point',
                            coordinates: center
                        },
                        place_name: feature.properties.display_name,
                        properties: feature.properties,
                        text: feature.properties.display_name,
                        place_type: ['place'],
                        center: center,
                    };
                    features.push(point);
                }
            } catch (e) {
                console.error(`Failed to forwardGeocode with error: ${e}`);
            }

            return {
                features: features
            };
        }
    };

    map.addControl(
        new MaplibreGeocoder(geocoder_api, {
            maplibregl: maplibregl,
            marker:false,
            draggable:true,
            color: "#FF0000"
        }),'top-left'

    );

    map.addControl(new NavigationControl(), 'top-right');
    // var marker = new Marker({color: "#FF0000", draggable:true})
    // .setLngLat([0,0])
    // .addTo(map);

    // var marker = new maplibregl.Marker({
    //     draggable:true
    // })
    // .setLngLat([0,0])
    // .addTo(map);


    // function add_maker (){
    //     var lngLat = new maplibregl.LngLat();
    //     console.log('Lng :', lngLat.lng, + 'Lat :', lngLat.lat);
    //     marker.setLngLat(lngLat).addTo(map);
    // }

    // map.on('click', add_maker);


    // function onDragEnd(){
    //   var lngLat = new maplibregl.getLngLat();
    //   coordinates.style.display = 'block';
    //   coordinates.innerHTML = 
    //   'Long : ' + lngLat.lng + '<br />Lat : ' + lngLat.lat
    // }
    // map.on('dragend', onDragEnd);



    // map.on('style.load', function(){
    //     map.on('click', function(e){
    //         var coordinates = e.lngLat;
    //         new maplibregl.Popup()
    //         .setLngLat(coordinates)
    //         .setHTML('Lng : ' + coordinates.lng + '<br />Lat : ' + coordinates.lat)
    //         .addTo(map)
    //     });
    // });
    
    map.on('style.load', function(){
        map.on('click', function(e){
            var coordinates = e.lngLat;
            new maplibregl.Marker({draggable:true})
            .setLngLat(coordinates)
            .addTo(map)
        });
    });

  });



</script>

<div class="map-wrap">
  <div class="map" id="map"></div>
  <!-- <div id="coordinates" class="coordinates"></div> -->
</div>

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
    z-index: 0;
  }
  .coordinates {
    background: gray;
    color: white;
    position: absolute;
    bottom:30px;
    font-size: 13px;
    z-index: 9999;
    display: none;
    padding: 5px;
    border-radius: 3px;
    left:10px;
    text-align: left;
  }
</style>