<script>
  import { userInfo } from "$lib/auth";

  import LinkButton from "$lib/components/link-button.svelte";

  export let model;
  export let secondary = false;
  export let inline = false;
  export let readOnly = true;
</script>

<div class:flex={inline} class:gap-s8={inline} class:items-start={inline}>
  {#if !readOnly}
    <LinkButton
      label="Modifier"
      to="/modeles/{model.slug}/editer"
      small
      noBackground={!secondary}
      {secondary}
    />
  {/if}

  {#if $userInfo}
    <LinkButton
      label="Créer un service"
      small
      to={`/services/creer?modele=${model.slug}`}
      noBackground={!secondary}
      {secondary}
    />
  {/if}
</div>
