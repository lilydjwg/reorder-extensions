<script lang="ts">
  let extmap
  let state
  let extlist = []
  let parsed_state
  let parsed_map = new Map()

  function update_state() {
    try{
      parsed_state = JSON.parse(state)
      extlist = parsed_state.placements['unified-extensions-area'].map(
        (x, i) => ({id: i, name: parsed_map.get(x), key: x})
      )
      console.log('extlist', extlist)
    }catch(e){
      extlist = []
    }
  }

  $: {
    try{
      parsed_map = new Map()
      for(let [k, v] of Object.entries(JSON.parse(extmap))) {
        parsed_map.set(k.replace(/[^A-Za-z0-9-]/g, '_') + '-browser-action', v)
      }
      update_state()
    }catch(e){
    }
  }

  let hovering = false;

  const drop = (event, target) => {
    event.dataTransfer.dropEffect = 'move'; 
    const start = parseInt(event.dataTransfer.getData("text/plain"));
    const newTracklist = extlist

    if (start < target) {
      newTracklist.splice(target + 1, 0, newTracklist[start]);
      newTracklist.splice(start, 1);
    } else {
      newTracklist.splice(target, 0, newTracklist[start]);
      newTracklist.splice(start + 1, 1);
    }
    parsed_state.placements['unified-extensions-area'] = newTracklist.map(o => o.key)
    extlist = newTracklist
    console.log('drop', extlist)
    hovering = null
  }

  const dragstart = (event, i) => {
    event.dataTransfer.effectAllowed = 'move';
    event.dataTransfer.dropEffect = 'move';
    const start = i;
    event.dataTransfer.setData('text/plain', start);
  }
</script>

<main>

<p>Open "about:debugging#/runtime/this-firefox", paste the following code into devtools console (F12) and copy the result:</p>
<pre><code>let data = {'{}'}
const section = document.querySelector('article &gt; section:nth-of-type(2)')
for(const ext of section.querySelectorAll('ul.debug-target-list &gt; li')){'{'}
  const title = ext.querySelector('[title]').title
  const uuid = ext.querySelector('section &gt; dl dd').textContent
  data[uuid] = title
{'}'}
console.log(JSON.stringify(data))
</code></pre>
<p>Paste the result string here:</p>
<textarea bind:value={extmap} on:input={update_state}></textarea>

<p>Open "about:config" and find "browser.uiCustomization.state". Copy its value here:</p>
<textarea bind:value={state} on:input={update_state}></textarea>

<p>Drag and drop to reorder:</p>
{#if extlist.length > 0}
  <div class="list">
  {#each extlist as n, index (n.key)}
    <div class="list-item" 
       draggable={true} 
       on:dragstart={event => dragstart(event, index)}
       on:drop|preventDefault={event => drop(event, index)}
       ondragover="return false"
       on:dragenter={() => hovering = index}
       class:is-active={hovering === index}>
       {n.name || "(deleted?)"}
    </div>
  {/each}
  </div>
  <p>Updated pref data:</p>
  <pre><code>{JSON.stringify(parsed_state)}</code></pre>
  <p>Copy and paste it back into the pref and restart Firefox. Note that not all extensions can be moved.</p>
{:else}
<p>Please paste data.</p>
{/if}

</main>

<style>
  pre {
    border: dashed gray 1px;
  }

  .list {
    background-color: white;
    border-radius: 4px;
    box-shadow: 0 2px 3px rgba(10, 10, 10, 0.1), 0 0 0 1px rgba(10, 10, 10, 0.1);
  }

  .list-item {
    display: block;
    padding: 0.5em 1em;
  }

  .list-item:not(:last-child) {
    border-bottom: 1px solid #dbdbdb;
  }

  .list-item.is-active {
    background-color: #3273dc;
    color: #fff;
  }
</style>
