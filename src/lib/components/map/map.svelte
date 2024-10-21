<script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster"; // Asegúrate de que esta librería esté correctamente cargada
  import "leaflet-rotatedmarker"; // Importa la clase para rotar el marcador (si lo necesitas para otras partes)

  let map: any;
  let userMarker: any;
  let userCoords = { lat: 0, lng: 0, h: 0 };
  let userHeading = 0;
  let directionCone: any; // Variable para el cono de dirección
  let loading = true; // Variable para el estado de carga

  // Generar lugares aleatorios cerca de la ubicación del usuario
  function generateRandomPlaces(lat: number, lng: number, count: number) {
    const places = [];
    const quarterKm = 0.0025; // Aproximadamente 1/4 de kilómetro en grados
    for (let i = 0; i < count; i++) {
      const randomLat = lat + (Math.random() - 0.5) * quarterKm; // Aproximadamente 1/4 km en latitud
      const randomLng = lng + (Math.random() - 0.5) * quarterKm; // Aproximadamente 1/4 km en longitud
      places.push({
        nombre: `Lugar ${i + 1}`,
        descripcion: `Descripción del lugar ${i + 1}.`,
        lat: randomLat,
        lng: randomLng,
      });
    }
    return places;
  }

  onMount(() => {
    // Esperar a que se obtenga la ubicación del usuario
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          userCoords.lat = position.coords.latitude;
          userCoords.lng = position.coords.longitude;

          // Mostrar mapa en la ubicación del usuario
          map = L.map("map").setView([userCoords.lat, userCoords.lng], 15.55);

          L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
            attribution: "© OpenStreetMap contributors",
          }).addTo(map);

          // Añadir marcador de ubicación del usuario (círculo azul)
          userMarker = L.circleMarker([userCoords.lat, userCoords.lng], {
            color: "#3388ff",
            fillColor: "#3388ff",
            fillOpacity: 0.5,
            radius: 10,
          }).addTo(map);

          // Generar lugares cercanos
          const lugares = generateRandomPlaces(
            userCoords.lat,
            userCoords.lng,
            5
          );

          lugares.forEach((lugar) => {
            const marker = L.marker([lugar.lat, lugar.lng]).addTo(map);
            marker.bindPopup(`<b>${lugar.nombre}</b><br>${lugar.descripcion}`);
          });

          // Verifica si hay orientación de dispositivo
          if (window.DeviceOrientationEvent) {
            window.addEventListener("deviceorientation", (event) => {
              if (event.alpha !== null) {
                userHeading = event.alpha; // Utiliza alpha para la dirección
                userCoords.h = userHeading;
                updateDirectionCone(
                  userCoords.lat,
                  userCoords.lng,
                  userHeading
                ); // Actualiza la dirección
              }
            });
          }

          loading = false; // Termina el estado de carga
        },
        () => {
          loading = false; // Termina el estado de carga en caso de error
          alert("No se pudo obtener la ubicación del usuario.");
        }
      );
    } else {
      loading = false; // Termina el estado de carga si la geolocalización no está disponible
      alert("Geolocalización no es soportada por este navegador.");
    }
  });

  function createDirectionCone(lat: number, lng: number, heading: number) {
    const coneLength = 10; // Longitud del cono
    const coneWidth = 6; // Ancho del cono

    // Calcular las coordenadas de los vértices del cono
    const point1 = L.latLng(lat, lng); // Punto de inicio (posición del usuario)
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

    // Crear el cono en el mapa
    directionCone = L.polygon(coneLatLngs, {
      color: "#FFD700", // Color amarillo
      fillColor: "#FFD700", // Color amarillo con opacidad
      fillOpacity: 0.3, // Opacidad del relleno
    }).addTo(map);
  }

  function updateDirectionCone(lat: number, lng: number, heading: number) {
    // Eliminar el cono de dirección anterior
    if (directionCone) {
      map.removeLayer(directionCone);
    }
    // Crear un nuevo cono de dirección
    createDirectionCone(lat, lng, heading);
  }
  
  $: console.log(userCoords);
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>
<div class="flex flex-col gap-[2px] my-2">
  <small>lat: {userCoords.lat}</small>
  <small>lng: {userCoords.lng}</small>
  <small>h: {userCoords.h}</small>
</div>

<!-- Mostrar cargando -->
{#if loading}
  <div class="text-center text-lg">Cargando mapa y lugares...</div>
{/if}

<div id="map" style="height: 500px;"></div>


<!-- <script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster"; // Asegúrate de que esta librería esté correctamente cargada
  import "leaflet-rotatedmarker"; // Importa la clase para rotar el marcador (si lo necesitas para otras partes)

  let map: any;
  let userMarker: any;
  let userCoords = { lat: 0, lng: 0 };
  let userHeading = 0;
  let directionCone: any; // Variable para el cono de dirección

  const lugares = [
    { nombre: "Lugar 1", descripcion: "Este es el lugar 1.", lat: 18.7710778, lng: -98.5441889 },
    { nombre: "Lugar 2", descripcion: "Este es el lugar 2.", lat: 18.7709122, lng: -98.5431473 },
    { nombre: "Lugar 3", descripcion: "Este es el lugar 3.", lat: 18.770548, lng: -98.542081 },
    { nombre: "Lugar 4", descripcion: "Este es el lugar 4.", lat: 18.771912, lng: -98.543291 },
    { nombre: "Lugar 5", descripcion: "Este es el lugar 5.", lat: 18.772281, lng: -98.545231 },
  ];

  onMount(() => {
    map = L.map("map").setView([18.770748, -98.542181], 15.55);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap contributors",
    }).addTo(map);

    lugares.forEach((lugar) => {
      const marker = L.marker([lugar.lat, lugar.lng]).addTo(map);
      marker.bindPopup(`<b>${lugar.nombre}</b><br>${lugar.descripcion}`);
    });

    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        userCoords.lat = position.coords.latitude;
        userCoords.lng = position.coords.longitude;

        // Añadir marcador de ubicación del usuario (círculo azul)
        userMarker = L.circleMarker([userCoords.lat, userCoords.lng], {
          color: "#3388ff",
          fillColor: "#3388ff",
          fillOpacity: 0.5,
          radius: 10,
        }).addTo(map);

        // Establecer la vista del mapa en la ubicación del usuario
        map.setView([userCoords.lat, userCoords.lng], 15.55);

        // Crear el cono de dirección
        // createDirectionCone(userCoords.lat, userCoords.lng, userHeading);

        // Verifica si hay orientación de dispositivo
        if (window.DeviceOrientationEvent) {
          window.addEventListener("deviceorientation", (event) => {
            if (event.alpha) {
              userHeading = event.alpha;
              updateDirectionCone(userCoords.lat, userCoords.lng, userHeading); // Actualiza la dirección
            }
          });
        }
      });
    }
  });

  function createDirectionCone(lat: number, lng: number, heading: number) {
    const coneLength = 10; // Longitud del cono
    const coneWidth = 6; // Ancho del cono

    // Calcular las coordenadas de los vértices del cono
    const point1 = L.latLng(lat, lng); // Punto de inicio (posición del usuario)
    const point2 = L.latLng(lat + (coneLength * Math.sin(heading * (Math.PI / 180))), lng + (coneLength * Math.cos(heading * (Math.PI / 180))));
    const point3 = L.latLng(lat + (coneWidth * Math.sin((heading + 45) * (Math.PI / 180))), lng + (coneWidth * Math.cos((heading + 45) * (Math.PI / 180))));
    const point4 = L.latLng(lat + (coneWidth * Math.sin((heading - 45) * (Math.PI / 180))), lng + (coneWidth * Math.cos((heading - 45) * (Math.PI / 180))));

    const coneLatLngs = [point1, point2, point3, point2, point4];

    // Crear el cono en el mapa
    directionCone = L.polygon(coneLatLngs, {
      color: "#FFD700", // Color amarillo
      fillColor: "#FFD700", // Color amarillo con opacidad
      fillOpacity: 0.3, // Opacidad del relleno
    }).addTo(map);
  }

  function updateDirectionCone(lat: number, lng: number, heading: number) {
    // Eliminar el cono de dirección anterior
    if (directionCone) {
      map.removeLayer(directionCone);
    }
    // Crear un nuevo cono de dirección
    createDirectionCone(lat, lng, heading);
  }
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>

<div id="map"></div>

<style>
  #map {
    height: 70vh;
  }
</style>
 -->

<style>
  #map {
    height: 70vh;
  }
</style>
