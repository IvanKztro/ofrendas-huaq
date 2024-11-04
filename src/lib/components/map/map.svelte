<script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster";
  import "leaflet-rotatedmarker";
  import { locations } from "$lib/stores/ofrendas";
  import Spinner from "./spinner.svelte";

  let map: any;
  let userMarker: any;
  let userCoords = { lat: 0, lng: 0 };
  let ofrendas: any = [];
  let informationstate: any = [];
  let parking: any = [];
  let hotels: any = [];
  let markers: any = [];
  let autoCenter = true; // Nueva bandera para controlar el centrado automático
  let selectedType = "all";
  let isLoading = true; // Estado de carga

  onMount(() => {
    // Establece el tiempo para el loading y espera que el DOM esté listo
    setTimeout(() => {
      ofrendas = $locations.filter((ll) => ll.type === "ofrenda");
      informationstate = $locations.filter((ll) => ll.type === "information");
      parking = $locations.filter((ll) => ll.type === "parking");
      hotels = $locations.filter((ll) => ll.type === "hotel");

      map = L.map("map").setView([18.770748, -98.542181], 15.55);

      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(map);

      // Desactivar centrado automático si el usuario interactúa con el mapa
      map.on("dragstart", () => {
        autoCenter = false;
      });

      // Obtener la geolocalización del usuario y mostrar los marcadores
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
          isLoading = false; // Oculta el loading cuando se cargan los markers
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
            isLoading = false; // Oculta el loading incluso si hay un error
          },
          {
            enableHighAccuracy: true,
            maximumAge: 0,
            timeout: 5000,
            distanceFilter: 1,
          }
        );
      }
    }, 0);
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

<h3 class="text-center text-black text-2xl">Ofrendas</h3>

<!-- Loading indicator -->
{#if isLoading}
<div class="flex items-center justify-center mt-[80px] h-[40dvh]">
  <Spinner></Spinner>

</div>
{:else}
  <div class="my-4 w-full flex gap-4 items-center">
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
      <button
        on:click={centerOnUser}
        class=" p-[8px] bg-blue-500 text-white rounded"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          class="lucide lucide-locate-fixed"
          ><line x1="2" x2="5" y1="12" y2="12" /><line
            x1="19"
            x2="22"
            y1="12"
            y2="12"
          /><line x1="12" x2="12" y1="2" y2="5" /><line
            x1="12"
            x2="12"
            y1="19"
            y2="22"
          /><circle cx="12" cy="12" r="7" /><circle
            cx="12"
            cy="12"
            r="3"
          /></svg
        >
      </button>
  </div>
{/if}
<div class="{isLoading ? 'invisible' : ' visible'}">
  <div id="map" ></div>

</div>

<style>
  #map {
    height: 70vh;
  }
  .loading {
    font-size: 1.5em;
    margin: 20px;
  }
</style>
