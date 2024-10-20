<script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster";
  import "leaflet-rotatedmarker";

  let map: any;
  let userMarker, directionMarker: any;
  let userCoords = { lat: 0, lng: 0 };
  let userHeading = 0;
  let loading = true; // Estado de carga

  // Función para generar ubicaciones aleatorias alrededor de la ubicación del usuario
  function generateRandomLocations(lat: number, lng: number, numLocations: number, radius: number) {
    const locations = [];

    for (let i = 0; i < numLocations; i++) {
      const angle = Math.random() * Math.PI * 2;
      const distance = Math.random() * radius;

      const deltaLat = distance * Math.cos(angle) / 111.32;
      const deltaLng = distance * Math.sin(angle) / (111.32 * Math.cos(lat * Math.PI / 180));

      const randomLat = lat + deltaLat;
      const randomLng = lng + deltaLng;

      locations.push({
        nombre: `Lugar Aleatorio ${i + 1}`,
        descripcion: `Este es el lugar aleatorio ${i + 1}.`,
        lat: randomLat,
        lng: randomLng,
      });
    }

    return locations;
  }

  onMount(() => {
    // Inicializar el mapa pero no establecer la vista aún
    map = L.map("map");

    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        userCoords.lat = position.coords.latitude;
        userCoords.lng = position.coords.longitude;

        // Establecer la vista del mapa en la ubicación del usuario
        map.setView([userCoords.lat, userCoords.lng], 15.55);

        // Ocultar el indicador de carga cuando la ubicación se obtiene
        loading = false;

        // Añadir marcador de ubicación del usuario (círculo azul)
        userMarker = L.circleMarker([userCoords.lat, userCoords.lng], {
          color: "#3388ff",
          fillColor: "#3388ff",
          fillOpacity: 0.5,
          radius: 10,
        }).addTo(map);

        // Cargar las capas de mapa
        L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
          attribution: "© OpenStreetMap contributors",
        }).addTo(map);

        // Crear ícono personalizado con una imagen de flecha
        const userDirectionIcon = L.icon({
          iconUrl: 'eyearea.png',
          iconSize: [20, 20],
          iconAnchor: [20, 20],
        });

        // Añadir marcador de orientación (flecha)
        directionMarker = L.marker([userCoords.lat, userCoords.lng], {
          icon: userDirectionIcon,
          rotationAngle: 0,
        }).addTo(map);

        // Generar 7 ubicaciones aleatorias dentro de un radio de 1 km alrededor de la ubicación del usuario
        const randomLocations = generateRandomLocations(userCoords.lat, userCoords.lng, 7, 1);

        // Agregar las ubicaciones aleatorias al mapa
        randomLocations.forEach((lugar) => {
          const marker = L.marker([lugar.lat, lugar.lng]).addTo(map);
          marker.bindPopup(`<b>${lugar.nombre}</b><br>${lugar.descripcion}`);
        });
      });

      if (window.DeviceOrientationEvent) {
        window.addEventListener("deviceorientation", (event) => {
          if (event.alpha) {
            userHeading = event.alpha;
            if (directionMarker) {
              directionMarker.setRotationAngle(userHeading);
            }
          }
        });
      }
    }
  });
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>

<!-- Mostrar el mensaje de carga mientras se obtiene la ubicación -->
{#if loading}
  <div class="loading-indicator">
    <p class="text-center text-gray-500">Obteniendo tu ubicación...</p>
  </div>
{/if}

<!-- Mapa -->
<div id="map" ></div>

<style>
  #map {
    height: 70vh;
  }

  .loading-indicator {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .hidden {
    display: none;
  }
</style>



<!-- <script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster";
  import "leaflet-rotatedmarker"; // Asegura que esta librería esté correctamente cargada

  let map: any;
  let userMarker, directionMarker: any;
  let userCoords = { lat: 0, lng: 0 };
  let userHeading = 0;

  const lugares = [
    {
      nombre: "Lugar 1",
      descripcion: "Este es el lugar 1.",
      lat: 18.7710778,
      lng: -98.5441889,
    },
    {
      nombre: "Lugar 2",
      descripcion: "Este es el lugar 2.",
      lat: 18.7709122,
      lng: -98.5431473,
    },
    {
      nombre: "Lugar 3",
      descripcion: "Este es el lugar 3.",
      lat: 18.770548,
      lng: -98.542081,
    },
    {
      nombre: "Lugar 4",
      descripcion: "Este es el lugar 4.",
      lat: 18.771912,
      lng: -98.543291,
    },
    {
      nombre: "Lugar 5",
      descripcion: "Este es el lugar 5.",
      lat: 18.772281,
      lng: -98.545231,
    },
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

        map.setView([userCoords.lat, userCoords.lng], 15.55);

        // Crear ícono personalizado con una imagen de flecha
        const userDirectionIcon = L.icon({
          iconUrl: 'eyearea.png', // Ruta a la imagen de la flecha
          iconSize: [20, 20], // Tamaño de la imagen
          iconAnchor: [20, 20], // Ancla para centrar la imagen en el marcador
        });

        // Añadir marcador de orientación (flecha)
        directionMarker = L.marker([userCoords.lat, userCoords.lng], {
          icon: userDirectionIcon,
          rotationAngle: 0, // Inicialmente sin rotación
        }).addTo(map);
      });

      if (window.DeviceOrientationEvent) {
        window.addEventListener("deviceorientation", (event) => {
          if (event.alpha) {
            userHeading = event.alpha;
            if (directionMarker) {
              // Rotar la imagen de la flecha según la orientación del dispositivo
              directionMarker.setRotationAngle(userHeading);
            }
          }
        });
      }
    }
  });
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>

<div id="map"></div>

<style>
  #map {
    height: 70vh;
  }
</style> -->


