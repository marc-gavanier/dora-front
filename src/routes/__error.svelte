<script context="module">
  export function load({ error, status }) {
    return {
      props: {
        notFound: status === 404,
        forbidden: status === 403,
        status,
        error,
      },
    };
  }
</script>

<script>
  import { onMount } from "svelte";
  import { trackError } from "$lib/utils/plausible";

  import { logException } from "$lib/logger";
  import CenteredGrid from "$lib/components/layout/centered-grid.svelte";

  export let status, error, notFound, forbidden;

  onMount(() => {
    trackError(`${status}`, document.location.pathname);

    if (!notFound) {
      const exc = new Error(error.message);
      exc.stack = error.stack;
      logException(exc, { error });
    }
  });
</script>

<svelte:head>
  <title>Erreur | DORA</title>
</svelte:head>

<CenteredGrid>
  <p>
    {#if notFound}
      Cette page n’existe pas.
    {:else if forbidden}
      Accès réservé
    {:else}
      Erreur inattendue.
    {/if}
  </p>
</CenteredGrid>
