<script>
  import CenteredGrid from "$lib/components/layout/centered-grid.svelte";
  import connexionPic from "$lib/assets/illu-connexion.svg";
</script>

<CenteredGrid bgColor="bg-magenta-10">
  <div class="flex flex-col gap-s24 lg:flex-row">
    <div class="mt-s48 hidden justify-center lg:flex lg:w-1/2">
      <img src={connexionPic} alt="" class="max-h-[460px]" />
    </div>
    <div class="lg:w-1/2">
      <slot />
    </div>
  </div>
</CenteredGrid>
