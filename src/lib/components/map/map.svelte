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
  let autoCenter = true;
  let selectedType = "all";
  let isLoading = true; // Bandera de carga

  onMount(() => {
    ofrendas = $locations.filter((ll) => ll.type === "ofrenda");
    informationstate = $locations.filter((ll) => ll.type === "information");
    parking = $locations.filter((ll) => ll.type === "parking");
    hotels = $locations.filter((ll) => ll.type === "hotel");

    // Inicializar el mapa pero aún no mostrarlo
    map = L.map("map").setView([18.770748, -98.542181], 15.55);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
    }).addTo(map);

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
        isLoading = false; // Ocultar el loading una vez que se cargan los marcadores
      });

      navigator.geolocation.watchPosition(
        (position) => {
          userCoords.lat = position.coords.latitude;
          userCoords.lng = position.coords.longitude;

          if (userMarker) {
            userMarker.setLatLng([userCoords.lat, userCoords.lng]);

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

  const getIconForLocation = (type) => {
    return L.icon({
      iconUrl: type === "ofrenda" ? "/ofrenda.gif" : `/${type}.png`,
      iconSize: type === "parking" ? [35, 25] : [30, 30],
      iconAnchor: [12, 25],
    });
  };

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

  const centerOnUser = () => {
    autoCenter = true;
    if (userMarker) {
      map.setView([userCoords.lat, userCoords.lng], 15.55);
    }
  };
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>
{#if isLoading}
  <div class="loading-spinner">
    <p>Cargando mapa y ubicaciones...</p>
    <!-- Puedes usar una animación de carga aquí -->
  </div>
{:else}
  <div class="my-4 w-full flex gap-2 items-center">
    <select
      bind:value={selectedType}
      on:change={() => showMarkers(selectedType)}
      class="p-2 rounded-lg border border-gray-300"
    >
      <option value="all">Todos</option>
      <option value="ofrenda">Ofrendas ({ofrendas?.length || 0})</option>
      <option value="information"
        >Información ({informationstate?.length || 0})</option
      >
      <option value="parking">Estacionamiento ({parking?.length || 0})</option>
      <option value="hotel">Hotel ({hotels?.length || 0})</option>
    </select>
    <button on:click={centerOnUser} class="ml-4 p-2 bg-blue-500 text-white rounded">
      <!-- Icono de centrar posición -->
    </button>
  </div>
  <div id="map"></div>
{/if}

<style>
  #map {
    height: 70vh;
  }

  .loading-spinner {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 70vh;
    font-size: 1.5rem;
    color: #333;
  }
</style>
