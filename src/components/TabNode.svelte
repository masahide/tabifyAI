<script lang="ts">
  interface Tab {
    id: number;
    title: string;
    url: string;
    favIconUrl?: string;
    openerTabId?: number;
    children?: Tab[];
  }
  export let tab: Tab;

  const toggle = () => {
    open = !open;
  };

  let open = true;
  function openTab(tabId: number) {
    chrome.tabs.update(tabId, { active: true });
  }
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<li class="tab-item" on:click={toggle}>
  {#if tab.favIconUrl}
    <img src={tab.favIconUrl} alt="favicon" />
  {/if}
  <!-- svelte-ignore missing-declaration -->
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <span on:click|stopPropagation={() => openTab(tab.id)}>{tab.title}</span>
</li>
{#if open && tab.children && tab.children.length > 0}
  <ul class="tab-children">
    {#each tab.children as child}
      <svelte:self {child} />
    {/each}
  </ul>
{/if}

<style>
  .tab-item {
    display: flex;
    align-items: center;
    cursor: pointer;
    padding: 5px;
  }
  .tab-item img {
    width: 16px;
    height: 16px;
    margin-right: 8px;
  }
  .tab-item:hover {
    background-color: #f0f0f0;
  }
  .tab-children {
    margin-left: 20px;
    padding-left: 10px;
    border-left: 1px solid #ddd;
  }
</style>
