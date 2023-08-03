<script>
  import { getContext, setContext, onMount, onDestroy  } from "svelte"
  import { writable } from "svelte/store"; 

  import Tab from "./lib/Tab.svelte"  
  export let size = "M"
  export let direction
  export let centered = false
  export let quiet = false
  export let compact = false
  export let emphasized = false
  export let emphasizedColor = "var(--primaryColor)"
  export let onTabChange
  export let initialIndex

  const { styleable, builderStore, componentStore, screenStore } = getContext("sdk")
  const component = getContext("component")

  let container
  let contents
  let vertical
  let tabItems = []
  let poller 

  // Set Store initial State
  let selectedTab = {
    id : undefined,
    name : "",
    boundingBox: null,
  }
  const tabStore = writable(selectedTab)
  setContext("tabStore", tabStore) 

  $: vertical = (direction === "vertical")

  $: populateTabsFromChildren($screenStore)
  $: hideNonSelected($tabStore.id)
  $: calculateIndicator($tabStore.id, vertical)

  // If in the builder preview, show this appropriate Container if a child is selected
  $: {
    if ( $builderStore.inBuilder && tabItems.length > 0 ) {
      let pos = tabItems?.findIndex( e => ( $componentStore?.selectedComponentPath.includes(e.id) ) )
      $tabStore.id = pos > -1 ? tabItems[pos].id : tabItems[0].id;  
      }     
    } 

  function populateTabsFromChildren ( full ) {
    if ( contents ) {
      if ( full || contents.children?.length != tabItems.length ) {
        tabItems = []
        for (let index = 0; index < contents.children.length; index++) {
          tabItems.push ({
            id: contents.children[index].attributes.getNamedItem("data-id")?.nodeValue,
            title: contents.children[index].attributes.getNamedItem("data-name")?.nodeValue
          })
          $tabStore.id = initialIndex < tabItems.length ? tabItems[initialIndex].id : tabItems[0].id
        }
      }
    }
  }

  function hideNonSelected () {
    if (contents?.children.length > 0) {
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
  }

  let left, top, width, height
  async function calculateIndicator ()  
  {
    if ($tabStore.id) {
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
  }

  function handleTabChange ( e ) {
    let context = { "tabName": e.detail.tabName }
    onTabChange?.( context )
  }
  
  onMount ( () => {
    poller = setInterval ( () => populateTabsFromChildren() , 50 )
  })

  onDestroy ( () => clearInterval(poller) )
</script>

<div use:styleable={$component.styles}>
 
    {#if $component?.children != 0 }
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
          {#each tabItems as _tab }
            <Tab 
              on:tabSelect={handleTabChange}
              title = {_tab.title} 
              isSelected = { $tabStore.id === _tab.id }
              id = {_tab.id} {emphasized}/>
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
      <div class="instructions"> 
          <h2>  ⚡️ Super Tabs Component </h2>
          <p> The Super Tabs Component will render a Tabs control with a Tab for each of it's child components </p>
          <p> It will use the "Name" property of the component as Tab text </p>
          <p> Add a couple of Container Components to get started ! </p>
      </div>
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