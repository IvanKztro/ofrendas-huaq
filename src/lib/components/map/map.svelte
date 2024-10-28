<script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster";
  import "leaflet-rotatedmarker";
  import { locations } from "$lib/stores/ofrendas";

  let map: any;
  let userMarker: any;
  let userCoords = { lat: 0, lng: 0 };
  let ofrendas: any = [];
  let informationstate: any = [];
  let parking: any = [];
  let hotels: any = [];
  let markers: any = [];
  let autoCenter = true;  // Nueva bandera para controlar el centrado automático

  let selectedType = "all";

  onMount(() => {
    ofrendas = $locations.filter((ll) => ll.type === "ofrenda");
    informationstate = $locations.filter((ll) => ll.type === "information");
    parking = $locations.filter((ll) => ll.type === "parking");
    hotels = $locations.filter((ll) => ll.type === "hotel");
    map = L.map("map").setView([18.770748, -98.542181], 15.55);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

    // Desactivar centrado automático si el usuario interactúa con el mapa
    map.on("dragstart", () => {
      autoCenter = false;
    });

    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        userCoords.lat = position.coords.latitude;
        userCoords.lng = position.coords.longitude;

        userMarker = L.circleMarker([userCoords.lat, userCoords.lng], {
          color: "#3388ff",
          fillColor: "#3388ff",
          fillOpacity: 0.7,
          radius: 10,
        }).addTo(map);

        map.setView([userCoords.lat, userCoords.lng], 15.55);
        showMarkers(selectedType);
      });

      navigator.geolocation.watchPosition(
        (position) => {
          userCoords.lat = position.coords.latitude;
          userCoords.lng = position.coords.longitude;

          // Actualizar la posición del marcador del usuario (círculo azul)
          if (userMarker) {
            userMarker.setLatLng([userCoords.lat, userCoords.lng]);

            // Solo centrar el mapa si autoCenter es true
            if (autoCenter) {
              map.panTo([userCoords.lat, userCoords.lng], { animate: true });
            }
          }
        },
        (error) => {
          console.error("Error en la geolocalización:", error);
        },
        {
          enableHighAccuracy: true,
          maximumAge: 0,
          timeout: 5000,
          distanceFilter: 1,
        }
      );
    }
  });

  // Función para obtener el icono de acuerdo al tipo
  const getIconForLocation = (type) => {
    return L.icon({
      iconUrl: type === "ofrenda" ? "/ofrenda.gif" : `/${type}.png`,
      iconSize: type === "parking" ? [35, 25] : [30, 30],
      iconAnchor: [12, 25],
    });
  };

  // Función para mostrar los marcadores filtrados
  const showMarkers = (type) => {
    markers.forEach((marker) => marker.remove());
    markers = [];

    $locations
      .filter((lugar) => type === "all" || lugar.type === type)
      .forEach((lugar) => {
        const marker = L.marker([lugar.lat, lugar.lng], {
          icon: getIconForLocation(lugar.type),
        }).addTo(map);
        marker.bindPopup(`<b>${lugar.type}: ${lugar.id}</b><br>${lugar.name}`);
        markers.push(marker);
      });
  };

  // Función para centrar manualmente el mapa en la posición del usuario
  const centerOnUser = () => {
    autoCenter = true;
    if (userMarker) {
      map.setView([userCoords.lat, userCoords.lng], 15.55);
    }
  };
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>
<div class="my-4 w-full">
  <select
    bind:value={selectedType}
    on:change={() => showMarkers(selectedType)}
    class="p-2 rounded-lg border border-gray-300"
  >
    <option value="all">Todos</option>
    <option value="ofrenda">Ofrendas ({ofrendas?.length || 0})</option>
    <option value="information">Información ({informationstate?.length || 0})</option>
    <option value="parking">Estacionamiento ({parking?.length || 0})</option>
    <option value="hotel">Hotel ({hotels?.length || 0})</option>
  </select>
  <!-- Botón para centrar en la posición del usuario -->
  <!-- <button on:click={centerOnUser} class="ml-4 p-2 bg-blue-500 text-white rounded">
    Centrar en mi posición
  </button> -->
</div>

<div id="map"></div>


<style>
  #map {
    height: 70vh;
  }
</style>



