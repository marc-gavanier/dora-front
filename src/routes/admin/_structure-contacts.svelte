<script>
  import EmailLine from "./_email-line.svelte";

  import InfoLine from "./_info-line.svelte";
  import UserInfo from "./_user-info.svelte";

  export let structure;
  const administrators = structure.members.filter((m) => m.isAdmin);
</script>

<InfoLine condition={structure.phone || structure.email}>
  {#if structure.phone}📞 {structure.phone}{/if}
  {#if structure.email}<EmailLine email={structure.email} />{/if}
</InfoLine>

<InfoLine>
  créée par: <UserInfo user={structure.creator} {structure} />
</InfoLine>

<InfoLine condition={structure.creator?.email !== structure.lastEditor?.email}>
  dernière modification par: <UserInfo
    user={structure.lastEditor}
    {structure}
  />
</InfoLine>

<div class="ml-s16">
  {#if administrators.length}
    <h6>administrateurs•trices</h6>
    {#each administrators as administrator}
      <InfoLine>
        <UserInfo user={administrator.user} {structure} />
      </InfoLine>
    {/each}
  {/if}
</div>
