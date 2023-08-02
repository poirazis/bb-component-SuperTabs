<script>
    import { getContext , createEventDispatcher, onMount } from "svelte"
    const dispatch = createEventDispatcher();

    const tabStore = getContext ("tabStore")
    export let id
    export let title
    export let emphasized

    let container
    export let isSelected

    $: init (id, title, emphasized, $tabStore.id, isSelected)

    function handleClick () {
        $tabStore.id = id
        $tabStore.boundingBox = container?.getBoundingClientRect();
        dispatch("tabSelect", {"tabID": id, "tabName": title } )
    }
    function init() {
      if (isSelected)
        $tabStore.boundingBox = container?.getBoundingClientRect();
    }

    onMount( () => init() )

</script>

<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div 
    bind:this={container}
    class="spectrum-Tabs-item"
    class:is-selected={isSelected}
    class:emphasized={isSelected && emphasized}
    tabindex="0"
    on:click={handleClick}>
        <span class="spectrum-Tabs-itemLabel"> {title} </span>
</div>

<style>
    .emphasized {
      color: var(--emphasizedColor) !important;
    }
    .spectrum-Tabs-item {
      color: var(--spectrum-global-color-gray-600);
    }
    .spectrum-Tabs-item.is-selected {
      color: var(--spectrum-global-color-gray-900);
    }
</style>