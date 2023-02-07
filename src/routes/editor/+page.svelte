<script lang="ts">
  import Icon from "../Icon.svelte";
  import { Trace, TLine } from "trace-svelte";

  // editor variables
  let content = "";
  let editor_step = 0;
  let editor_highlighted : number[] = [];

  // Trace component variables
  let step = 0;
  let highlights : number[][] = [[]];
  let playing = false;
  let playInterval : ReturnType<typeof setInterval>;
  
  // for preview
  function increment() { if (step < highlights.length - 1) { step += 1; } } 
  function decrement() { if (step != 0) { step -= 1; } }
  function togglePlaying() { playing = !playing; }
  function autoplay() {
    if (step < highlights.length - 1) { increment(); }
    else {
      togglePlaying();
      step = 0;
    }
  }
  
  // for editor
  function editorIncrement() { 
    if (editor_step < highlights.length - 1) { 
      editor_step += 1; 
    } 
    updateEditorHighlighted(); 
  } 

  function editorDecrement() { 
    if (editor_step != 0) {
      editor_step -= 1; 
    } 
    updateEditorHighlighted(); 
  }

  function updateEditorHighlighted() { editor_highlighted = highlights[editor_step]; }

  function addStep() { 
    highlights = [...highlights, []];
    editor_step = highlights.length - 1;
  }

  function toggleLine(line : number) { 
    if (editor_highlighted.includes(line + 1)) { 
      editor_highlighted = editor_highlighted.filter((l) => l != (line + 1)); 
    }
    else { editor_highlighted = [...editor_highlighted, line + 1]; }
  }
  

  $: lines = content.split("\n");
  $: highlights[editor_step] = editor_highlighted;
  $: {
    if (playing) { playInterval = setInterval(autoplay, 1000); }
    else { clearInterval(playInterval); }
  }
</script>

<main class="flex flex-row gap-8 w-screen h-screen p-16 bg-base">
  <div class="flex flex-col gap-8 w-full basis-1/2">
    <div class="flex flex-row gap-8 justify-start">
      <p class="text-2xl text-text font-bold">Editor</p>
      <button on:click={editorDecrement} class="text-text">
        <Icon name="arrow-small-left" size={24} />
      </button>
      <p class="text-lg text-text">{editor_step + 1}</p>
      <button on:click={editorIncrement} class="text-text">
        <Icon name="arrow-small-right" size={24} />
      </button>
      <button on:click={addStep} class="text-text">
        <Icon name="plus" size={24} />
      </button>
    </div>
    <div class="flex flex-row">
      <div class="flex flex-col p-8 gap-1">
        {#each lines as _, i}
          <button on:click={() => toggleLine(i)}> 
            {#if editor_highlighted.includes(i+1)}
              <p class="text-md text-text">⬤</p>
            {:else}
              <p class="text-md text-crust">⬤</p>
            {/if}
          </button>
        {/each}
      </div>
      <textarea 
        bind:value={content}
        class="bg-overlay0 overflow-y-clip p-8 w-full h-full text-xl rounded-md" 
      /> 
    </div>
  </div>
  <div class="flex flex-col h-full overflow-y-auto basis-1/2 bg-crust rounded-md p-4">
    <div class="flex flex-row justify-between w-full h-fit px-8 py-4">
      <div class="flex flex-row gap-8">
        <p class="text-xl text-text font-bold">Preview</p>
        <button on:click={togglePlaying} class="text-text">
          {#if playing}
            <Icon name="pause" size={24} />
          {:else}
            <Icon name="play" size={24} />
          {/if}
        </button>
        <button on:click={decrement} class="text-md text-text">
          <Icon name="arrow-small-left" size={24} />
        </button>
        <button on:click={increment} class="text-text">
          <Icon name="arrow-small-right" size={24} />
        </button>
      </div>
      <p class="text-lg text-text diagonal-fractions">
        {step + 1}/{highlights.length}
      </p>
    </div>
    <div class="w-full h-fit border-text border-2 rounded-md">
      <Trace {step} {highlights}>
      
        {#each lines as line}
          <TLine>
            <p class="text-xl text-[#cdd6f4]">
              {line}
            </p>
          </TLine>
        {/each}

      </Trace>
    </div>
  </div>
</main>



