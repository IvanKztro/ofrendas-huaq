<script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster";
  import "leaflet-rotatedmarker";
  import { locations } from "$lib/stores/ofrendas";

  let map: any;
  let userMarker: any;
  let userCoords = { lat: 0, lng: 0 };
  let userHeading = 0;
  let directionCone: any;

  let ofrendas: any = [];
  let informationstate: any = [];
  let parking: any = [];
  let hotels: any = [];
  let markers: any = [];

  let selectedType = "all";

  onMount(() => {
    ofrendas = $locations.filter((ll) => ll.type === "ofrenda");
    informationstate = $locations.filter((ll) => ll.type === "information");
    parking = $locations.filter((ll) => ll.type === "parking");
    hotels = $locations.filter((ll) => ll.type === "hotel");
    map = L.map("map").setView([18.770748, -98.542181], 15.55);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap contributors",
    }).addTo(map);

    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        userCoords.lat = position.coords.latitude;
        userCoords.lng = position.coords.longitude;

        userMarker = L.circleMarker([userCoords.lat, userCoords.lng], {
          color: "#3388ff",
          fillColor: "#3388ff",
          fillOpacity: 0.5,
          radius: 10,
        }).addTo(map);

        map.setView([userCoords.lat, userCoords.lng], 15.55);
        showMarkers(selectedType);
      });

      navigator.geolocation.watchPosition((position) => {
        userCoords.lat = position.coords.latitude;
        userCoords.lng = position.coords.longitude;

        if (userMarker) {
          userMarker.setLatLng([userCoords.lat, userCoords.lng]);
          map.setView([userCoords.lat, userCoords.lng], map.getZoom()); // Opcional: Centra el mapa en la nueva ubicación
        }

        if (window.DeviceOrientationEvent) {
          window.addEventListener("deviceorientation", (event) => {
            if (event.alpha !== null) {
              userHeading = event.alpha;
              updateDirectionCone(userCoords.lat, userCoords.lng, userHeading);
            }
          });
        }
      });
    }
  });

  const getIconForLocation = (type) => {
    return L.icon({
      iconUrl: type === "ofrenda" ? "/ofrenda.gif" : `/${type}.png`,
      iconSize: [25, 25],
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

  function createDirectionCone(lat: number, lng: number, heading: number) {
    const coneLength = 10;
    const coneWidth = 6;

    const point1 = L.latLng(lat, lng);
    const point2 = L.latLng(
      lat + coneLength * Math.sin(heading * (Math.PI / 180)),
      lng + coneLength * Math.cos(heading * (Math.PI / 180))
    );
    const point3 = L.latLng(
      lat + coneWidth * Math.sin((heading + 45) * (Math.PI / 180)),
      lng + coneWidth * Math.cos((heading + 45) * (Math.PI / 180))
    );
    const point4 = L.latLng(
      lat + coneWidth * Math.sin((heading - 45) * (Math.PI / 180)),
      lng + coneWidth * Math.cos((heading - 45) * (Math.PI / 180))
    );

    const coneLatLngs = [point1, point2, point3, point2, point4];

    directionCone = L.polygon(coneLatLngs, {
      color: "#FFD700",
      fillColor: "#FFD700",
      fillOpacity: 0.3,
    }).addTo(map);
  }

  function updateDirectionCone(lat: number, lng: number, heading: number) {
    if (directionCone) {
      directionCone.setLatLngs(createConePoints(lat, lng, heading));
    } else {
      createDirectionCone(lat, lng, heading);
    }
  }

  function createConePoints(lat, lng, heading) {
    const coneLength = 10;
    const coneWidth = 6;
    const point1 = L.latLng(lat, lng);
    const point2 = L.latLng(
      lat + coneLength * Math.sin(heading * (Math.PI / 180)),
      lng + coneLength * Math.cos(heading * (Math.PI / 180))
    );
    const point3 = L.latLng(
      lat + coneWidth * Math.sin((heading + 45) * (Math.PI / 180)),
      lng + coneWidth * Math.cos((heading + 45) * (Math.PI / 180))
    );
    const point4 = L.latLng(
      lat + coneWidth * Math.sin((heading - 45) * (Math.PI / 180)),
      lng + coneWidth * Math.cos((heading - 45) * (Math.PI / 180))
    );
    return [point1, point2, point3, point2, point4];
  }
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>
<div class="my-4 w-full">
  <select
    bind:value={selectedType}
    on:change={() => showMarkers(selectedType)}
    class="p-2 rounded-lg border border-gray-300"
  >
    <option value="all">Mostrar Todos</option>
    <option value="ofrenda">Ofrendas ({ofrendas?.length || 0})</option>
    <option value="information">Información ({informationstate?.length || 0})</option>
    <option value="parking">Estacionamiento ({parking?.length || 0})</option>
    <option value="hotel">Hotel ({hotels?.length || 0})</option>
  </select>
</div>

<div id="map"></div>

<style>
  #map {
    height: 70vh;
  }
</style>



<!-- <script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster";
  import "leaflet-rotatedmarker";
  import { locations } from "$lib/stores/ofrendas";

  let map: any;
  let userMarker: any;
  let userCoords = { lat: 0, lng: 0 };
  let userHeading = 0;
  let directionCone: any;

  let ofrendas: any = [];
  let informationstate: any = [];
  let parking: any = [];
  let hotels: any = [];
  let markers: any = [];

  let selectedType = "all";

  onMount(() => {
    //   // Filtrar ubicaciones por tipo
    ofrendas = $locations.filter((ll) => ll.type === "ofrenda");
    informationstate = $locations.filter((ll) => ll.type === "information");
    parking = $locations.filter((ll) => ll.type === "parking");
    hotels = $locations.filter((ll) => ll.type === "hotel");
    map = L.map("map").setView([18.770748, -98.542181], 15.55);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap contributors",
    }).addTo(map);

    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        userCoords.lat = position.coords.latitude;
        userCoords.lng = position.coords.longitude;

        userMarker = L.circleMarker([userCoords.lat, userCoords.lng], {
          color: "#3388ff",
          fillColor: "#3388ff",
          fillOpacity: 0.5,
          radius: 10,
        }).addTo(map);

        map.setView([userCoords.lat, userCoords.lng], 15.55);
        showMarkers(selectedType);
      });

      // Usar watchPosition para actualizar la posición del usuario sin generar nuevos puntos
      navigator.geolocation.watchPosition((position) => {
        userCoords.lat = position.coords.latitude;
        userCoords.lng = position.coords.longitude;

        // Actualizar la posición del marcador del usuario (círculo azul)
        if (userMarker) {
          userMarker.setLatLng([userCoords.lat, userCoords.lng]);
        }

        // Actualizar el cono de dirección si el dispositivo tiene orientación
        if (window.DeviceOrientationEvent) {
          window.addEventListener("deviceorientation", (event) => {
            if (event.alpha) {
              userHeading = event.alpha;
              updateDirectionCone(userCoords.lat, userCoords.lng, userHeading);
            }
          });
        }
      });
    }
  });

  // Función para obtener el icono de acuerdo al tipo
  const getIconForLocation = (type) => {
    return L.icon({
      iconUrl: type === "ofrenda" ? "/ofrenda.gif" : `/${type}.png`,
      iconSize: [25, 25],
      iconAnchor: [12, 25],
    });
  };

  // Función para mostrar los marcadores filtrados
  const showMarkers = (type) => {
    // Eliminar marcadores existentes
    markers.forEach((marker) => marker.remove());
    markers = [];

    // Filtrar y añadir nuevos marcadores según el tipo
    $locations
      .filter((lugar) => type === "all" || lugar.type === type)
      .forEach((lugar) => {
        const marker = L.marker([lugar.lat, lugar.lng], {
          icon: getIconForLocation(lugar.type),
        }).addTo(map);
        marker.bindPopup(`<b>${lugar.type}: ${lugar.id}</b><br>${lugar.name}`);
        markers.push(marker); // Guardar el marcador en la lista para controlarlo después
      });
  };

  function createDirectionCone(lat: number, lng: number, heading: number) {
    const coneLength = 10;
    const coneWidth = 6;

    const point1 = L.latLng(lat, lng);
    const point2 = L.latLng(
      lat + coneLength * Math.sin(heading * (Math.PI / 180)),
      lng + coneLength * Math.cos(heading * (Math.PI / 180))
    );
    const point3 = L.latLng(
      lat + coneWidth * Math.sin((heading + 45) * (Math.PI / 180)),
      lng + coneWidth * Math.cos((heading + 45) * (Math.PI / 180))
    );
    const point4 = L.latLng(
      lat + coneWidth * Math.sin((heading - 45) * (Math.PI / 180)),
      lng + coneWidth * Math.cos((heading - 45) * (Math.PI / 180))
    );

    const coneLatLngs = [point1, point2, point3, point2, point4];

    directionCone = L.polygon(coneLatLngs, {
      color: "#FFD700",
      fillColor: "#FFD700",
      fillOpacity: 0.3,
    }).addTo(map);
  }

  function updateDirectionCone(lat: number, lng: number, heading: number) {
    if (directionCone) {
      map.removeLayer(directionCone);
    }
    createDirectionCone(lat, lng, heading);
  }
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>
<div class="my-4 w-full">
  <select
    bind:value={selectedType}
    on:change={() => showMarkers(selectedType)}
    class="p-2 rounded-lg border border-gray-300"
  >
    <option value="all">Mostrar Todos</option>
    <option value="ofrenda">Ofrendas ({ofrendas?.length || 0})</option>
    <option value="information"
      >Información ({informationstate?.length || 0})</option
    >
    <option value="parking">Estacionamiento ({parking?.length || 0})</option>
    <option value="hotel">Hotel ({hotels?.length || 0})</option>
  </select>
</div>

<div id="map"></div>

<style>
  #map {
    height: 70vh;
  }
</style> -->
