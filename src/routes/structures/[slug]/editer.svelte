<script context="module">
  import { getStructuresOptions } from "$lib/structures";

  export async function load() {
    return {
      props: {
        structuresOptions: await getStructuresOptions(),
      },
    };
  }
</script>

<script>
  import { getStructure } from "$lib/structures";
  import EnsureLoggedIn from "$lib/components/ensure-logged-in.svelte";
  import StructureFormWrapper from "$lib/components/structures/form-wrapper.svelte";
  import { structure } from "./_store";
  import { refreshUserInfo } from "$lib/auth";
  import { capitalize } from "$lib/utils";

  export let structuresOptions;

  async function handleRefresh() {
    $structure = await getStructure($structure.slug);

    await refreshUserInfo();
  }
</script>

<svelte:head>
  <title>
    {capitalize($structure.name)} | Éditer | DORA
  </title>
</svelte:head>

<EnsureLoggedIn>
  <h2 class="mb-s40 border-b border-gray-02 pb-s40">Informations</h2>

  <StructureFormWrapper
    structure={$structure}
    {structuresOptions}
    modify
    onRefresh={handleRefresh}
  />
</EnsureLoggedIn>
