<script lang="ts">
  import Button from "$lib/components/button.svelte";

  import type { Service, ServicesOptions } from "$lib/types";
  import SubcategoryListItem from "./subcategory-list-item.svelte";

  export let service: Service;
  export let servicesOptions: ServicesOptions;

  const numberOfSubcategories = 3;
  let expanded = false;

  $: subcategories = expanded
    ? service.subcategories
    : service.subcategories.slice(0, numberOfSubcategories);
</script>

<ul>
  {#each subcategories as subCategory, i (subCategory)}
    <li class="mb-s8">
      <SubcategoryListItem subCategorySlug={subCategory} {servicesOptions} />
    </li>
  {/each}
</ul>

{#if !expanded && service.subcategories.length > numberOfSubcategories}
  <Button
    label="Voir tous les besoins"
    on:click={() => (expanded = true)}
    noBackground
    small
    noPadding
    hoverUnderline
  />
{/if}
