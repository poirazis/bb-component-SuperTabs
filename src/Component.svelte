<script>
  import { getContext, setContext, onMount } from "svelte"
  import { writable } from "svelte/store"; 
  import { findComponentById } from "./lib/helpers"
  import Tab from "./lib/Tab.svelte"
  
  export let size = "M"
  export let direction
  export let centered = false
  export let quiet = false
  export let compact = false
  export let emphasized = false
  export let emphasizedColor = "var(--primaryColor)"

  const { styleable, builderStore, screenStore, componentStore } = getContext("sdk")
  const component = getContext("component")

  // 
  let container
  let contents
  let vertical
  let _tabs = undefined
  let _fallbackSelectedID = undefined

  // Set Store initial State
  let selectedTab = {
    id : undefined,
    boundingBox: null,
  }
  const tabStore = writable(selectedTab)
  setContext("tabStore", tabStore) 

  $: vertical = (direction === "vertical")

  $: if ($screenStore.activeScreen.props && $component.id) {
    let node = findComponentById($screenStore.activeScreen.props, $component.id)
    if (node) {
      _tabs = node._children;
      _fallbackSelectedID = _tabs[0]?._id;
    }
  }

  $: if (!$tabStore.id) $tabStore.id = _fallbackSelectedID
  $: hideNonSelected($tabStore.id)
  $: calculateIndicator($tabStore.id, vertical)

  // If in the builder preview, show this appropriate Container if a child is selected
  $: {
    if ( $builderStore.inBuilder && _tabs.length > 0 ) {
      let pos =  _tabs?.findIndex( e => ( $componentStore?.selectedComponentPath.includes(e._id) ) )
      $tabStore.id = pos > -1 ? _tabs?.[pos]._id : _fallbackSelectedID;  
     }     
  } 

  function hideNonSelected () {
    let children = contents?.children

    for(var i=0; i<children?.length; i++){
      var child = children[i];
      let id = (child.attributes.getNamedItem("data-id")?.nodeValue);   
      if (id != $tabStore?.id) 
        child.style.display = "none"
      else 
        child.style.display = "block"
    }   
  }

  let left, top, width, height
  function calculateIndicator ()  
  {
      if (!vertical) {
        width = $tabStore?.boundingBox?.width + "px"
        height = "2px";
        top =  $tabStore?.boundingBox?.height + "px"
        left = $tabStore?.boundingBox?.left - container?.getBoundingClientRect().left + "px"
      } else {
        height = $tabStore?.boundingBox?.height + 4 + "px"
        width = "2px"
        top = $tabStore?.boundingBox?.top - container?.getBoundingClientRect().top + "px"
        left = -2 + "px"
      }
  }

  onMount ( () => {
    hideNonSelected(); 
    calculateIndicator();
  })

  $: console.log ($tabStore.id)
</script>

<div use:styleable={$component.styles}>
 
    {#if _tabs?.length > 0 && $tabStore.id }
    <div 
      bind:this={container} 
      class="wrapper" 
      class:vertical={vertical}
      style="--emphasizedColor: {emphasizedColor};"
    > 
      <div 
        class:centered={!vertical && centered}
        class:spectrum-Tabs--quiet={quiet}
        class:spectrum-Tabs--vertical={vertical}
        class:spectrum-Tabs--horizontal={!vertical}
        class:spectrum-Tabs--compact={compact}
        class="spectrum-Tabs spectrum-Tabs--size{size}"
      >
          {#each _tabs as _tab}
            <Tab title = {_tab._instanceName} id = {_tab._id} {emphasized}/>
          {/each}

         <div 
          class="spectrum-Tabs-selectionIndicator"
          class:emphasized = {emphasized} 
          style="width: {width}; height: {height}; left: {left}; top: {top};">
        </div> 
      </div>

      <div bind:this={contents} class="tabContents"> 
        <slot />
      </div>

    </div>
    {:else}
      <p> Please add some containers to get started ! </p>
    {/if}

</div>

<style>
  .tabContents {
    width: 100%;
    flex-grow: 1 0;
  }
  .spectrum-Tabs--quiet {
    border-bottom: none !important;
  }
  .spectrum-Tabs {
      padding-left: var(--spacing-xl);
      padding-right: var(--spacing-xl);
      position: relative;
      border-bottom-color: var(--spectrum-global-color-gray-200);
      border-left-color: var(--spectrum-global-color-gray-200);
  }

  .wrapper {
    display: flex;
    flex-direction: column;
    width: 100%;
    gap: 0.85em;
  }
  .vertical {
    display: flex !important;
    flex-direction: row !important;
  }
  .centered {
    justify-content: center !important;
  }
  .spectrum-Tabs-content {
    margin-top: var(--spectrum-global-dimension-static-size-150);
  }
  .spectrum-Tabs--vertical.spectrum-Tabs--compact {
    border-left-color: var(--spectrum-global-color-gray-200);
  }

  .spectrum-Tabs-selectionIndicator {
    transition: all 300ms;
    background-color: var(--spectrum-global-color-gray-900);
  }

  .spectrum-Tabs--vertical.spectrum-Tabs--compact .spectrum-Tabs-selectionIndicator {
    background-color: var(--spectrum-global-color-gray-900);
  }
  .spectrum-Tabs-selectionIndicator.emphasized {
    background-color: var(--emphasizedColor) !important;
  }

  .noHorizPadding {
    padding: 0;
  }
  .noPadding {
    margin: 0;
  }
  .onTop {
    z-index: 100;
  }
</style>