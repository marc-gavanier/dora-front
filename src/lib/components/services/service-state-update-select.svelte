<script lang="ts">
  // Source pour l'accessibilité : https://www.w3.org/WAI/ARIA/apg/example-index/combobox/combobox-select-only.html

  import { goto } from "$app/navigation";
  import { uid } from "uid";

  import {
    archiveService,
    convertSuggestionToDraft,
    deleteService,
    getService,
    publishService,
    unarchiveService,
    unPublishService,
  } from "$lib/services";

  import {
    SERVICE_STATUSES,
    type DashboardService,
    type Service,
  } from "$lib/types";
  import { getAvailableOptionsForStatus } from "$lib/utils/service";
  import { validate } from "$lib/validation";
  import { serviceSchema } from "$lib/schemas/service.js";
  import {
    arrowDownSIcon,
    arrowUpSIcon,
    deleteBinIcon,
    earthFillIcon,
    fileEditFillIcon,
    folderFillIcon,
    draftFillIcon,
  } from "$lib/icons";
  import { clickOutside } from "../use/click-outside";

  type ServiceStatusPresentation = {
    bgClass: string;
    iconClass: string;
    hoverBgClass: string;
    icon: string;
    label: string;
  };

  export const SERVICE_STATUS_PRESENTATION: Record<
    SERVICE_STATUSES,
    ServiceStatusPresentation
  > = {
    [SERVICE_STATUSES.SUGGESTION]: {
      bgClass: "bg-service-violet",
      iconClass: "text-service-violet-darker",
      hoverBgClass: "bg-service-violet-dark",
      icon: fileEditFillIcon,
      label: "Suggestion",
    },
    [SERVICE_STATUSES.DRAFT]: {
      bgClass: "bg-service-orange",
      iconClass: "text-service-orange-darker",
      hoverBgClass: "bg-service-orange-dark",
      icon: draftFillIcon,
      label: "Brouillon",
    },
    [SERVICE_STATUSES.PUBLISHED]: {
      bgClass: "bg-service-green",
      iconClass: "text-service-green-darker",
      hoverBgClass: "bg-service-green-dark",
      icon: earthFillIcon,
      label: "Publié",
    },
    [SERVICE_STATUSES.ARCHIVED]: {
      bgClass: "bg-service-gray",
      iconClass: "text-gray-darker",
      hoverBgClass: "bg-service-gray-dark",
      icon: folderFillIcon,
      label: "Archivé",
    },
  };

  export let service: Service | DashboardService;
  export let servicesOptions;
  export let onRefresh: () => void;
  export let hideLabel = true;
  export let fullWidth = false;

  // *** Valeurs pour l'affichage
  $: currentStatusPresentation = SERVICE_STATUS_PRESENTATION[service.status];
  $: availableOptions = getAvailableOptionsForStatus(service.status);

  // *** Accessibilité
  const uuid: string = uid(); // Pour éviter les conflits d'id si le composant est présent plusieurs fois sur la page
  let isDropdownOpen = false;

  // Gestion de l'outline avec la navigation au clavier
  let selectedOptionIndex: number | null = null;
  let selectedOption: SERVICE_STATUSES | "DELETE" | null = null;

  function toggleCombobox(forceValue?: boolean) {
    isDropdownOpen = forceValue !== undefined ? forceValue : !isDropdownOpen;
    if (isDropdownOpen) {
      selectedOptionIndex = 0;
      selectedOption = availableOptions[selectedOptionIndex];
    } else {
      selectedOptionIndex = null;
      selectedOption = null;
    }
  }

  function handleKeydown(event: KeyboardEvent) {
    if (["Escape", " ", "Enter", "ArrowDown", "ArrowUp"].includes(event.key)) {
      event.preventDefault();
    }
    if (event.code === "Tab" || event.code === "Escape") toggleCombobox(false);

    if (event.code === "Space") toggleCombobox();

    if (event.code === "Enter") {
      if (!isDropdownOpen) {
        toggleCombobox(true);
      } else {
        if (selectedOption) {
          updateServiceStatus(selectedOption);
        }
      }
    }

    if (event.code === "ArrowDown") {
      selectedOptionIndex = (selectedOptionIndex || 0) + 1;
      selectedOptionIndex = selectedOptionIndex % availableOptions.length;

      selectedOption = availableOptions[selectedOptionIndex];
    }
    if (event.code === "ArrowUp") {
      selectedOptionIndex = (selectedOptionIndex || 0) - 1;
      if (selectedOptionIndex < 0)
        selectedOptionIndex = availableOptions.length - 1;
      selectedOption = availableOptions[selectedOptionIndex];
    }
  }

  function setAsSelected(
    hoveredStatus: SERVICE_STATUSES | "DELETE",
    index: number
  ) {
    selectedOptionIndex = index;
    selectedOption = hoveredStatus;
  }

  // Actions disponibles
  async function updateServiceStatus(newStatus: SERVICE_STATUSES | "DELETE") {
    if (newStatus === SERVICE_STATUSES.DRAFT) {
      if (service.status === SERVICE_STATUSES.SUGGESTION) {
        await convertSuggestionToDraft(service.slug);
      } else if (service.status === SERVICE_STATUSES.PUBLISHED) {
        await unPublishService(service.slug);
      } else if (service.status === SERVICE_STATUSES.ARCHIVED) {
        await unarchiveService(service.slug);
      }
    } else if (newStatus === SERVICE_STATUSES.PUBLISHED) {
      await publish();
    } else if (newStatus === SERVICE_STATUSES.ARCHIVED) {
      await archiveService(service.slug);
    } else if (newStatus === "DELETE") {
      // eslint-disable-next-line no-alert
      if (confirm(`Supprimer la suggestion ${service.name} ?`)) {
        await deleteService(service.slug);
        goto(`/structures/${service.structure}/`);
      }
    }

    toggleCombobox(false);
    if (onRefresh) await onRefresh();
  }

  async function publish() {
    let serviceFull = service;
    // teste si on a le service complet
    // ça n'est pas le cas sur les cards de la page structure par exemple

    if (!Object.prototype.hasOwnProperty.call(service, "fullDesc")) {
      serviceFull = await getService(service.slug);
    }

    const isValid = validate(serviceFull, serviceSchema, {
      noScroll: true,
      showErrors: false,
      extraData: servicesOptions,
    }).valid;

    if (isValid) {
      await publishService(service.slug);
    } else {
      goto(`/services/${service.slug}/editer`);
    }
  }
</script>

<div
  id="service-state-update"
  class="relative flex cursor-pointer items-center rounded-md font-bold text-gray-dark {currentStatusPresentation.bgClass} hover:{currentStatusPresentation.hoverBgClass}"
  use:clickOutside
  on:click_outside={() => toggleCombobox(false)}
>
  <span id={`button-label-${uuid}`} class="sr-only">
    Modifier le status du service
  </span>

  <div
    aria-controls={`listbox-values-${uuid}`}
    aria-expanded={isDropdownOpen}
    aria-haspopup="listbox"
    aria-labelledby={`button-label-${uuid}`}
    id={`update-listbox-${uuid}`}
    class="cursor:pointer flex items-center px-s20 py-s10"
    role="combobox"
    tabindex="0"
    on:click={() => toggleCombobox()}
    on:keydown={handleKeydown}
  >
    <span class:hidden={hideLabel} class="mr-s10">Statut du service :</span>

    <span
      class="{currentStatusPresentation.iconClass} mr-s8 h-s24 w-s24 fill-current"
    >
      {@html currentStatusPresentation.icon}
    </span>

    <span>{currentStatusPresentation.label}</span>
    <span class="ml-s10 h-s24 w-s24 fill-current text-magenta-cta">
      {#if isDropdownOpen}
        {@html arrowUpSIcon}
      {:else}
        {@html arrowDownSIcon}
      {/if}
    </span>
  </div>

  <div
    class:hidden={!isDropdownOpen}
    class:w-full={fullWidth}
    class="absolute top-s48 right-s0 z-20 min-w-[150px] rounded border border-gray-00 bg-white py-s12 px-s12 shadow-md"
    role="listbox"
    id={`listbox-values-${uuid}`}
    aria-labelledby={`button-label-${uuid}`}
    tabindex="-1"
  >
    {#each availableOptions as option, index (option)}
      {#if option === "DELETE"}
        <div
          class="mb-s10 flex items-center rounded bg-transparent p-s10"
          class:bg-service-red={selectedOption === option}
          on:mouseenter={() => setAsSelected(option, index)}
          role="option"
          on:click={() => updateServiceStatus(option)}
        >
          <span class="mr-s8 h-s24 w-s24 fill-current text-service-red-dark">
            {@html deleteBinIcon}
          </span>
          <span>Supprimer</span>
        </div>
      {:else}
        {@const data = SERVICE_STATUS_PRESENTATION[option]}
        <div
          class="mb-s10 flex items-center rounded p-s10 {selectedOption ===
          option
            ? data.hoverBgClass
            : 'bg-transparent'}"
          role="option"
          id={option}
          on:mouseenter={() => setAsSelected(option, index)}
          on:click={() => updateServiceStatus(option)}
        >
          <span class="{data.iconClass} mr-s8 h-s24 w-s24 fill-current">
            {@html data.icon}
          </span>
          <span>{data.label}</span>
        </div>
      {/if}
    {/each}
  </div>
</div>
