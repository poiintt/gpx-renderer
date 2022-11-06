<script lang="ts">
  import { gpx } from "@tmcw/togeojson";
  import { geoMercator } from "d3-geo";

  let files: FileList;
  const width = 800;
  const height = 600;

  async function getPath(file: File) {
    const xml = await file.text();
    const document = new DOMParser().parseFromString(xml, "text/xml");
    const featureCollection = gpx(document);

    if ("coordinates" in featureCollection.features[0].geometry) {
      const coordinates = featureCollection.features[0].geometry
        .coordinates as [number, number, number][];

      const padding = 20;
      const geoProjection = geoMercator().fitExtent(
        [
          [padding, padding],
          [width - padding * 2, height - padding * 2],
        ],
        featureCollection
      );

      const mappedCoordinates = coordinates.map((cord) =>
        geoProjection([cord[0], cord[1]])
      );
      const path = "M " + mappedCoordinates.join(" L ");
      return path;
    }
    return "";
  }
</script>

<label for="route">Upload a GPX file:</label>
<input accept=".gpx" bind:files id="route" name="route" type="file" />

{#if files}
  <h2>Selected files:</h2>
  {#each Array.from(files) as file}
    <p>{file.name} ({file.size} bytes)</p>

    {#await getPath(file)}
      <p>...loading map</p>
    {:then d}
      <svg {width} {height}>
        <path
          {d}
          fill="none"
          stroke="red"
          stroke-width="2"
          stroke-linecap="round"
          stroke-miterlimit="4"
        />
      </svg>
    {:catch error}
      <p style="color: red">{error.message}</p>
    {/await}
  {/each}
{/if}
