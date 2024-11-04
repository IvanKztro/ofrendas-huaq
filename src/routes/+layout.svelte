<script>
  import Footer from "$lib/components/footer.svelte";
  import "../app.css";
</script>

<!-- <div class="app bg-slate-300">
  <main>
    <slot />
  </main>
</div> -->

<div class=" min-h-screen app bg-slate-300">
  <slot />
  <div class="fixed bottom-0 w-full z-50">
    <Footer className="" />
  </div>
</div>

<style>
  .app {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
  }

  main {
    flex: 1;
    display: flex;
    flex-direction: column;
    padding: 1rem;
    width: 100%;
    max-width: 64rem;
    margin: 0 auto;
    box-sizing: border-box;
  }

  footer {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 12px;
  }

  footer a {
    font-weight: bold;
  }

  @media (min-width: 480px) {
    footer {
      padding: 12px 0;
    }
  }
</style>
