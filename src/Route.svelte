<script lang="ts">
  import { gpx } from "@tmcw/togeojson";
  import { geoMercator } from "d3-geo";
  import type { Position } from "geojson";

  import { humanFileSize } from "./utils";

  export let file: File = undefined;
  const width = 800;
  const height = 600;
  let d = "M ";
  let drawProgress = 0;

  function isPositionArray(
    position: Position | Position[] | Position[][] | Position[][][]
  ): position is Position[] {
    const first = (position as Position[])[0];
    return typeof first[0] === "number" && typeof first[1] === "number";
  }

  function getProgress(value: number, total: number) {
    const result = (value + 1) / total;
    return (result * 100) | 0;
  }

  async function getPath(file: File) {
    const xml = await file.text();
    const document = new DOMParser().parseFromString(xml, "text/xml");
    const featureCollection = gpx(document);

    const geometry = featureCollection.features[0].geometry;
    if ("coordinates" in geometry) {
      const coordinates = geometry.coordinates;

      if (isPositionArray(coordinates)) {
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

        const coordinateCount = mappedCoordinates.length;
        let offset = (coordinateCount / 1000) | 0;
        offset = offset > 5 ? offset : 5;

        function extendPath(i: number) {
          drawProgress = getProgress(i, coordinateCount);
          if (i < coordinateCount) {
            const coordinates = mappedCoordinates.slice(i, i + offset);
            d += ` ${coordinates}`;

            i += offset;
            setTimeout(() => {
              extendPath(i);
            });
          }
        }

        extendPath(0);
      }
    }
  }

  getPath(file);
</script>

<p>{file.name} ({humanFileSize(file.size)})</p>
{#if drawProgress < 100}
  <p>{drawProgress}%</p>
{/if}
<svg {width} {height}>
  <path
    {d}
    fill="none"
    stroke="green"
    stroke-width="2"
    stroke-linecap="round"
    stroke-miterlimit="4"
  />
</svg>
