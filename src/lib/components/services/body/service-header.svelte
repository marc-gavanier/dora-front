<script lang="ts">
  import Breadcrumb from "$lib/components/breadcrumb.svelte";
  import type { Service } from "$lib/types";
  import { capitalize } from "$lib/utils";

  export let service: Service;
</script>

<div
  id="service-header"
  class="relative gap-s16 lg:flex-row-reverse lg:justify-between"
>
  <div class="mb-s48 print:mb-s0">
    <Breadcrumb
      {service}
      structure={service.structureInfo}
      currentLocation="service"
    />
  </div>
  <h1 class="mb-s14 leading-[3rem] text-white print:text-france-blue">
    {service.name}
  </h1>

  <div
    class="mb-s48 flex flex-col text-f18 text-white print:text-france-blue md:flex-row md:items-center"
  >
    <div><strong>{capitalize(service.structureInfo.name)}</strong></div>
    <div
      class="mx-s8 hidden font-bold print:hidden md:block print:md:hidden"
      aria-hidden="true"
    >
      •
    </div>
    <div class="print:hidden">
      <a
        class="underline"
        href="/structures/{service.structureInfo.slug}/services"
        >Voir les autres services ({service.structureInfo.numServices})</a
      >
    </div>
  </div>

  <div
    class="mb-s32 flex flex-col text-f18  text-white print:text-france-blue md:flex-row md:items-center"
  >
    <div>
      Périmètre : <strong>{service.diffusionZoneDetailsDisplay}</strong>
    </div>
  </div>
</div>
