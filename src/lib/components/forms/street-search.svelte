<script>
  import { getContext } from "svelte";

  import { contextValidationKey } from "$lib/validation";

  import Select from "$lib/components/forms/select.svelte";

  export let handleChange;
  export let placeholder;
  export let cityCode;
  export let disabled;
  export let name;
  export let value = undefined;
  export let initialValue = undefined;
  const banAPIUrl = "https://api-adresse.data.gouv.fr/search/";

  async function searchAddress(q) {
    const url = `${banAPIUrl}?q=${encodeURIComponent(
      q
    )}&limit=10&citycode=${cityCode}&type=street&type=housenumber`;
    const response = await fetch(url);
    const jsonResponse = await response.json();
    const results = jsonResponse.features.map((feature) => ({
      value: feature,
      label: feature.properties.name,
    }));
    return results;
  }

  const context = getContext(contextValidationKey);

  function handleBlur(evt) {
    if (context) context.onBlur(evt);
  }
</script>

<Select
  bind:value
  on:blur={handleBlur}
  {name}
  onChange={handleChange}
  {initialValue}
  {placeholder}
  {disabled}
  hideArrow
  searchFunction={searchAddress}
  delay="200"
  localFiltering={false}
  minCharactersToSearch="3"
/>
