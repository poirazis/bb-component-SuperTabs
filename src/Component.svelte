<script>
  import { getContext, setContext, onMount, tick  } from "svelte"
  import { writable } from "svelte/store"; 
  import { elementSizeStore } from "svelte-legos";

  import Tab from "./lib/Tab.svelte"  
  export let size = "M"
  export let direction
  export let centered = false
  export let quiet = false
  export let compact = false
  export let emphasized = false
  export let emphasizedColor = "var(--primaryColor)"
  export let onTabChange

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  let tabsToRender = []

  let container
  let contents
  let vertical
  let sizeStore
  let _fallbackSelectedID = undefined
  let children_no = 0

  $: console.log(children_no)

  // Set Store initial State
  let selectedTab = {
    id : undefined,
    name : "",
    boundingBox: null,
  }
  const tabStore = writable(selectedTab)
  setContext("tabStore", tabStore) 

  $: vertical = (direction === "vertical")


  $: populateTabsFromChildren( $sizeStore )
  $: hideNonSelected($tabStore.id)
  $: calculateIndicator($tabStore.id, vertical)

  function populateTabsFromChildren () {
    let contentChildren = contents?.children;
    if ( contentChildren && (contentChildren.length != children_no) ) { 
      console.log("Regenerating")
      tabsToRender = [];
      children_no = contentChildren.length;

      for ( let i = 0; i < contentChildren.length; i++ ) {
        let id = contentChildren[i].attributes.getNamedItem("data-id")?.nodeValue;
        let title = contentChildren[i].attributes.getNamedItem("data-name")?.nodeValue;
        tabsToRender.push({ id : id, title: title })
      }
      _fallbackSelectedID = tabsToRender[0].id;
      $tabStore.id = tabsToRender[0].id;
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
      console.log("Calculating")
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
    sizeStore = elementSizeStore(contents);
    populateTabsFromChildren();
  })

  $: console.log($sizeStore)
</script>

<div use:styleable={$component.styles}>
 
    {#if $$slots  }
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
          {#each tabsToRender as _tab}
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
          <h2> Super Tabs Component </h2>
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