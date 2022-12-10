<script>
    import { getContext } from "svelte"
    const tabStore = getContext ("tabStore")
    export let id
    export let title
    export let emphasized

    let container
    let isSelected

    $: isSelected = ( $tabStore.id === id )
    $: init (id, title, emphasized, $tabStore.id)

    function handleClick () {
        $tabStore.id = id
        $tabStore.boundingBox = container?.getBoundingClientRect();
    }
    function init() {
        if (isSelected)
            $tabStore.boundingBox = container?.getBoundingClientRect();
    }

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