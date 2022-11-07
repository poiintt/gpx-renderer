<script lang="ts">
  import Route from "./Route.svelte";

  let files: FileList;
</script>

<label for="route">Upload a GPX file:</label>
<input accept=".gpx" bind:files id="route" name="route" type="file" multiple />

{#if files}
  <h2>Selected files:</h2>
  {#each Array.from(files) as file}
    <Route {file} />
  {/each}
{/if}
