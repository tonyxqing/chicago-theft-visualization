<script lang="ts">
  import { each } from "svelte/internal";

    const TILE_DIM = 50;

    function MakeTiles(numTiles: number) {
        const grid = new Array();
        for (let i = 0; i < numTiles; i++) {
            grid.push([])
            for (let j = 0; j < numTiles; j++) {
                grid[i].push({x: 1+TILE_DIM * i, y: 1+TILE_DIM * j})
            }
        }
        return grid
    }
</script>

<g>
{#each MakeTiles(16) as row}
    <g>
    {#each row as col}
     <!-- content here -->
        <rect fill="white" stroke="rgba(1,1,1,.1)" x={col.x} y={col.y} width={TILE_DIM} height={TILE_DIM}></rect>
     {/each}
    </g>
{/each}
</g>