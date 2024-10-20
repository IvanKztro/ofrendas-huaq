<script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster";
  import "leaflet-rotatedmarker"; // Asegúrate de que esta librería esté correctamente cargada

  let map: any;
  let userMarker, directionMarker: any;
  let userCoords = { lat: 0, lng: 0 };
  let userHeading = 0;

  const lugares = [
    { nombre: "Lugar 1", descripcion: "Este es el lugar 1.", lat: 18.7710778, lng: -98.5441889 },
    { nombre: "Lugar 2", descripcion: "Este es el lugar 2.", lat: 18.7709122, lng: -98.5431473 },
    { nombre: "Lugar 3", descripcion: "Este es el lugar 3.", lat: 18.770548, lng: -98.542081 },
    { nombre: "Lugar 4", descripcion: "Este es el lugar 4.", lat: 18.771912, lng: -98.543291 },
    { nombre: "Lugar 5", descripcion: "Este es el lugar 5.", lat: 18.772281, lng: -98.545231 },
  ];

  let lightCone: any; // Variable para el cono de luz

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

        

        // Crear el cono de luz inicial en función de la orientación del dispositivo
        if (window.DeviceOrientationEvent) {
          window.addEventListener("deviceorientation", (event) => {
            if (event.alpha) {
              userHeading = event.alpha;
              directionMarker.setRotationAngle(userHeading); // Rotar la imagen de la flecha según la orientación del dispositivo
              updateLightCone(userCoords.lat, userCoords.lng, userHeading); // Actualizar el cono de luz
            }
          });
        }
        
        // Llama a la función para inicializar el cono de luz
        updateLightCone(userCoords.lat, userCoords.lng, userHeading);
      });
    }
  });

  function createLightCone(lat: number, lng: number, heading: number) {
    const coneLength = 10; // Longitud del cono (ajustado para ser más corto)
    const coneWidth = 10; // Ancho del cono (ajustado para ser igual al círculo)

    // Calcular las coordenadas de los vértices del cono
    const point1 = L.latLng(lat, lng);
    const point2 = L.latLng(lat + (coneLength * Math.sin(heading * (Math.PI / 180))), lng + (coneLength * Math.cos(heading * (Math.PI / 180))));
    const point3 = L.latLng(lat + (coneWidth * Math.sin((heading + 45) * (Math.PI / 180))), lng + (coneWidth * Math.cos((heading + 45) * (Math.PI / 180))));
    const point4 = L.latLng(lat + (coneWidth * Math.sin((heading - 45) * (Math.PI / 180))), lng + (coneWidth * Math.cos((heading - 45) * (Math.PI / 180))));

    const coneLatLngs = [point1, point2, point3, point2, point4];

    lightCone = L.polygon(coneLatLngs, {
      color: "#3388ff", // Color azul
      fillColor: "#3388ff", // Color azul con opacidad
      fillOpacity: 0.2, // Opacidad del relleno
    }).addTo(map);
  }

  function updateLightCone(lat: number, lng: number, heading: number) {
    // Eliminar el cono de luz anterior
    if (lightCone) {
      map.removeLayer(lightCone);
    }
    // Crear un nuevo cono de luz
    createLightCone(lat, lng, heading);
  }
</script>

<h3 class="text-center text-blue-500 text-2xl">Ofrendas Huaquechula</h3>

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

