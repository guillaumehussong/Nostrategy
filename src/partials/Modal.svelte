<script>
  import {randomId} from "hurdak/lib/hurdak"
  import {onMount, onDestroy} from "svelte"
  import {fly, fade} from "svelte/transition"
  import {modal} from "src/partials/state"

  export let virtual = true
  export let onEscape = null

  let root, content

  const id = randomId()

  onMount(() => {
    if (virtual) {
      modal.push({id, virtual: true})
    }
  })

  onDestroy(() => {
    if (virtual) {
      modal.remove(id)
    }
  })
</script>

<svelte:body
  on:keydown={e => {
    if (e.key === "Escape" && !root.querySelector(".modal")) {
      onEscape?.()
    }
  }} />

<div class="modal fixed inset-0 z-30" bind:this={root} transition:fade>
  <div
    class="fixed inset-0 cursor-pointer bg-black opacity-50"
    on:click|stopPropagation={onEscape} />
  <div
    class="modal-content h-full overflow-auto"
    bind:this={content}
    transition:fly={{y: 1000}}
    class:cursor-pointer={onEscape}
    on:click={onEscape}>
    <div class="mt-12 min-h-full">
      <div class="pointer-events-none sticky top-0 z-10 flex w-full justify-end p-2">
        <div
          class:opacity-0={!onEscape}
          class="pointer-events-auto flex h-10 w-10 cursor-pointer items-center justify-center
               rounded-full border border-solid border-accent-light bg-accent text-white">
          <i class="fa fa-times fa-lg" />
        </div>
      </div>
      <div class="absolute mt-12 h-full w-full bg-gray-7" />
      <div
        class="relative h-full w-full cursor-auto border-t border-solid border-gray-6 bg-gray-7 pt-2 pb-10"
        on:click|stopPropagation>
        <slot />
      </div>
    </div>
  </div>
</div>
