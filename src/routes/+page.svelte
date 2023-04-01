<script lang="ts">
	import { onMount } from 'svelte';
  import chicagodata from './../chicagodata.json';
  import kde_motor_theft from './../kde_motor_theft.json';
  import kde_theft from './../kde_theft.json';
  import chicago_outline from './../chicago_outline.json'
  import * as d3 from 'd3';
  import Scrolly from "./Scrolly.svelte";

  // Array values from Chicago Crime data
  // #11 lon 
  // #10 lat

    let value: number;
    const steps = [
      "<p>Last year in Chicago there were over .</p>",
      "<p>The scatterplot uses tweened values to automatically update your points with smooth transitions. It also binds to the width of the container <code>div</code>, so its responsive by default.</p>",
      "<p>Try resizing me to see the 'side-by-side' version, compared to the 'text-on-top' version that appears on small screens.</p><p>Want it to always appear 'text-on-top'? Just comment out the media query at the bottom of our styles (as in, leave the styles but comment out the surrounding <code>media</code> query).</p>",      "<p>Last year in Chicago there were over .</p>",
      "<p>The scatterplot uses tweened values to automatically update your points with smooth transitions. It also binds to the width of the container <code>div</code>, so its responsive by default.</p>",
      "<p>Try resizing me to see the 'side-by-side' version, compared to the 'text-on-top' version that appears on small screens.</p><p>Want it to always appear 'text-on-top'? Just comment out the media query at the bottom of our styles (as in, leave the styles but comment out the surrounding <code>media</code> query).</p>",
    ];
    kde_motor_theft.features.sort((d, k) => d.geometry.coordinates[0].length - k.geometry.coordinates[0].length) 
    kde_theft.features.sort((d, k) => d.geometry.coordinates[0].length - k.geometry.coordinates[0].length)

    const height = 800;
    const width = 800;
    const viewBox = `0 0 ${height} ${width}`;
    let projection: d3.GeoProjection;
    $: pathGenerator = d3.geoPath(projection);
    onMount(() => {
      projection = d3.geoMercator().scale((100 * width)).center([-87.6598, 41.9081])
    })
</script>
<section >
  <div class="hero">
    <h1>Chicago Data Visualization</h1>
    <h2>By <a href="https://tonyxqing.github.io" target="_blank">Tony</a></h2>
  </div>
<div class="section-container">
  <div class="steps-container">
    <Scrolly bind:value>
      {#each steps as text, i}
      <div class="step" class:active={value === i}>
        <div class="step-content">{@html text}</div>
      </div>
      {/each}
    </Scrolly>
  </div>
  <div class="map-container sticky">
    <svg {viewBox} >
        <clipPath id="clip-path">
          <path d={pathGenerator(chicago_outline.features[0])} fill="none" stroke="black" stroke-width="1px"></path>
        </clipPath>
        <g class="map">
          
          {#each kde_motor_theft.features as d, i}
            <path clip-path="url(#clip-path)"  id={`mt-${i}`} d={pathGenerator(d)} opacity={value === 3 ? 1 : 0} fill={`rgba(${45 * (i)},${15 * i},${5 * i},.25)`} stroke="black"></path>          
          {/each}
          {#each kde_theft.features as d, i}
            <path clip-path="url(#clip-path)" id={`t-${i}`} d={pathGenerator(d)} opacity={value === 4 ? 1 : 0} fill={`rgba(${45 * (i)},${15 * i},${1 * i}, .25)`} stroke="black"></path>
          {/each}

        </g>
        <g >
          {#each chicagodata.features as d, i}
          <!-- svelte-ignore a11y-click-events-have-key-events -->
            <path id={`clipPath`} d={pathGenerator(d)} fill={value === 0 ? "white" : 
                                                              value === 1 ? `rgba(244, 15, 151, ${d.properties.num_motor_thefts})` :
                                                              value === 2 ? `rgba(244, 15, 151, ${d.properties.num_thefts})` : "rgba(255,255,255,.1)"} stroke="black" on:click={(event) => {console.log(d.properties.pri_neigh)}}></path>
          
          {/each}
        </g>

    </svg>
  </div>
</div>
<div class="hero">
  <h1>Thanks for visiting!</h1>
  <h2><a href="https://github.com/tonyxqing" target="_blank">Github</a></h2>
</div>

  
</section>
<style>
  :global(body) {
    height: 300vh;
    font-family: "Comic Sans MS";
  }

  path {
    transition: 500ms;
  }

  .map-container {
    display: flex;
    height: 90vh;
    max-width: 100%;
    background: linear-gradient(to bottom right, steelblue -100%, white 100%);
    border-radius: 5px;
    box-shadow: 1px 1px 6px #cecece;
  }
  svg {
    border: 2px solid rgb(0, 0, 0);
    width: 100%;
    pointer-events: all;
  }
  path{
    transition: 500ms;
  }
  .hero {
    height: 30vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }

  .step {
    height: 100vh;
    display: flex;
    place-items: center;
    justify-content: center;
  }

  .step-content {
    font-size: 1rem;
    background: whitesmoke;
    color: #ccc;
    border-radius: 5px;
    padding: .5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 500ms ease;
    box-shadow: 1px 1px 10px rgba(0, 0, 0, .2);
    text-align: left;
		width: 75%;
		margin: auto;
		max-width: 500px;
  }
  .sticky {
    position: sticky;
    top: 2%;
    flex: 1 1 60%;
    width: 60%;
    margin: 1rem;
  }
  .steps-container{
    height: 100%;
  }

  .section-container {
    width: 100%;
    margin-top: 1em;
    text-align: center;
    transition: background 100ms;
    display: flex;
  }
  .steps-container {
    z-index: 10;

  }
	.step.active .step-content {
		background: white;
		color: black;
	}
  
  @media screen and (max-width: 768px) {
    .section-container {
      flex-direction: column-reverse;
    }
    .sticky {
      width: 95%;
			margin: auto;
    }
  }
</style>