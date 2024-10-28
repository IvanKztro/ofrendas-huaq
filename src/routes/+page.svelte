<script lang="ts">
  import Map from "$lib/components/map/map.svelte";
</script>

<svelte:head>
  <title>Mapa con Ofrendas</title>
  <meta name="description" content="Mapa con marcas y ubicación del usuario" />
</svelte:head>

<div class="">
  <Map></Map>
  <!-- <div class="h-5 bg-white  -mt-4 divclass relative"></div> -->
</div>

<style>
  .divclass{
    z-index: 9999;
  }
</style>