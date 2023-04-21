<script lang="ts">
  export let streams: MediaStream[] = [];
</script>

<div class="flex items-center gap-x-3 mt-2">
  <p class="text-2xl font-bold">Preview</p>
</div>
<div class="flex my-3 flex-wrap gap-2">
  {#each streams as stream}
    <div class="flex flex-col gap-1">
      <!-- svelte-ignore a11y-media-has-caption -->
      <video
        id={stream.id}
        class="w-96 h-72"
        playisinline
        autoplay
      />
    </div>
  {/each}
</div>
