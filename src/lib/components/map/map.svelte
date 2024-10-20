<script lang="ts">
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet.markercluster";
  import "leaflet-rotatedmarker";

  let map: any;
  let userMarker, directionMarker;
  let userCoords = { lat: 0, lng: 0 };
  let userHeading = 0;

  // Lista de lugares con coordenadas y descripción
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
    {
      nombre: "Lugar 6",
      descripcion: "Este es el lugar 6.",
      lat: 18.769912,
      lng: -98.541891,
    },
    {
      nombre: "Lugar 7",
      descripcion: "Este es el lugar 7.",
      lat: 18.769321,
      lng: -98.543701,
    },
    {
      nombre: "Lugar 8",
      descripcion: "Este es el lugar 8.",
      lat: 18.772711,
      lng: -98.546012,
    },
    {
      nombre: "Lugar 9",
      descripcion: "Este es el lugar 9.",
      lat: 18.769012,
      lng: -98.540912,
    },
    {
      nombre: "Lugar 10",
      descripcion: "Este es el lugar 10.",
      lat: 18.773112,
      lng: -98.543451,
    },
    {
      nombre: "Lugar 11",
      descripcion: "Este es el lugar 11.",
      lat: 18.772512,
      lng: -98.544712,
    },
    {
      nombre: "Lugar 12",
      descripcion: "Este es el lugar 12.",
      lat: 18.770211,
      lng: -98.544112,
    },
  ];

  onMount(() => {
    // Inicializa el mapa
    map = L.map("map").setView([18.770748, -98.542181], 15.55);

    // Añade las capas de OpenStreetMap
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap contributors",
    }).addTo(map);

    // Crea el grupo de clusters
    // const markerCluster = L.markerClusterGroup();

    // Muestra las marcas de los lugares
    lugares.forEach((lugar) => {
      const marker = L.marker([lugar.lat, lugar.lng]).addTo(map);
      marker.bindPopup(`<b>${lugar.nombre}</b><br>${lugar.descripcion}`);
      //   marker.addLayer(marker);
    });

    // Añade el grupo de clusters al mapa
    // map.addLayer(markerCluster);

    // Solicitar la ubicación del usuario
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        userCoords.lat = position.coords.latitude;
        userCoords.lng = position.coords.longitude;

        // Añadir marcador personalizado con un círculo
        userMarker = L.circleMarker([userCoords.lat, userCoords.lng], {
          color: "transparent",
          fillColor: "transparent",
          fillOpacity: 0.5,
          radius: 10,
        }).addTo(map);

        map.setView([userCoords.lat, userCoords.lng], 15.55);

        // Crear marcador de orientación (flecha)
        const userDirectionIcon = L.icon({
          iconUrl: "/arrow.png",
          iconSize: [30, 30],
          iconAnchor: [15, 20],
        });

        directionMarker = L.marker([userCoords.lat, userCoords.lng], {
          icon: userDirectionIcon,
          rotationAngle: userHeading,
          rotationOrigin: "center",
        }).addTo(map);
      });

      // Detectar orientación del dispositivo
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

<div id="map"></div>

<style>
  #map {
    height: 70vh;
  }
</style>
