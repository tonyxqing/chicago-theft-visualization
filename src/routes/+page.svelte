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
      "<p>As one of America's largest and most diverse cities, Chicago has long been a hub of activity for both residents and tourists alike. Unfortunately, it has also been a hotbed for theft and motor theft, with many citizens falling victim to these crimes every year.</p><p> In this data visualization, we will examine the prevalence and trends of theft and motor theft in Chicago, shedding light on this important issue and providing insights into the challenges faced by law enforcement and communities in combating these crimes.</p>",
      "<p>The choropleth map that we've created for this data visualization uses normalized data based on both the number of thefts and the area of each neighborhood in Chicago. The intensity of the color on the map reflects the relative severity of the problem in each neighborhood, with darker shades indicating a higher number of thefts per unit area.</p><p>By using this color scale, we can quickly identify the neighborhoods that are most affected by theft and motor theft in the city, highlighting areas that may require greater attention and resources from law enforcement and community organizations.</p>",
      "<p>In addition to the choropleth map, we've also used a kernel density plot to visualize the distribution of thefts across Chicago. Through providing a continuous representation of the density of thefts across the city, this plot can reveal patterns that might not be immediately apparent from other types of visualization. For instance, it can help identify areas with high concentrations of thefts that might not be highlighted by the choropleth map, which can be influenced by the size and shape of each neighborhood.</p><p>Using this complementary approach, we can gain a more complete understanding of the prevalence and distribution of theft in Chicago, and better inform efforts to address this important issue.</p>",
      "<p>Our choropleth map of motor thefts in Chicago provides an overview of the distribution of this type of crime across the city. While the map may not reveal many noticeable differences compared to the theft map, it does show that certain neighborhoods have a higher concentration of incidents relative to their size. We can see that the concentration for motor theft shifts primarily to South Chicago when compared to thefts.</p>",
      "<p>The KDE plot of motor thefts in Chicago highlights the difference in pattern of motor theft concentration compared to our previous plot of thefts overall. In contrast to the previous plot that showed the highest concentration of thefts occurring in the Loop and extending up to Rogers Park in the North, this plot shows that the highest concentration of motor thefts is clustered around the Loop and down to the South Chicago area.</p><p>This information can be useful for identifying the areas where interventions and prevention strategies need to be targeted to address the specific challenge of motor theft in Chicago. By examining both plots, we can develop a more complete understanding of the spatial patterns of theft and motor theft across the city.</p>",
      "<p>Visualizations of theft and motor theft in Chicago reveal that these crimes are prevalent across the city, with concentrations varying in different neighborhoods. Combating these crimes presents significant challenges for law enforcement and communities alike, including limited resources, complex socio-economic factors, and evolving criminal tactics.<p></p>Additionally, dense urban areas and underreporting of incidents further complicate efforts to address these crimes effectively. Despite these challenges, law enforcement and communities are working together to implement prevention and intervention strategies to reduce the incidence of theft and motor theft in Chicago. By utilizing data-driven approaches, such as the visualizations presented here, we can better understand the scope of the problem and allocate resources more effectively to combat these crimes.</p>"
    ];

    let tick = 0;
    setInterval(() => {
      tick += 1;
    }, 3000)
    const height = 800;
    const width = 800;
    const viewBox = `0 0 ${height} ${width}`;
    let projection: d3.GeoProjection = d3.geoMercator().scale((100 * width)).center([-87.6598, 41.9081]);;
    $: console.log(projection);
    $: tick = tick % kde_motor_theft.features.length;
    $: pathGenerator = d3.geoPath(projection);
    onMount(() => {
      projection = d3.geoMercator().scale((100 * width)).center([-87.6598, 41.9081]);
      pathGenerator = d3.geoPath(projection);
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
          <path d={pathGenerator(chicago_outline.features[0])} fill="white" stroke="black" stroke-width="1px"></path>
        </clipPath>
        <rect clip-path="url(#clip-path)" fill="white" height="100%" width="100%"></rect>
        <g >
          {#each chicagodata.features as d, i}
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          
            <path d={pathGenerator(d)} fill={value === 0 ? "white" : 
                                                              value === 3 || value === 4 ? `rgba(244, 15, 151, ${(d.properties.num_motor_thefts ?? 0) / (parseInt(d.properties.shape_area) * 0.00000000268936051 + .1)})` :
                                                              value === 1 || value === 2 ? `rgba(244, 15, 151, ${(d.properties.num_thefts ?? 0) / (parseInt(d.properties.shape_area) * 0.00000000268936051 + .1) })` : "none"} stroke="black" on:click={(event) => {console.log(d)}}></path>
          {/each}
        </g>
        <g class="map">
          {#if value === 4}
            {#each kde_motor_theft.features as d, i}
            <path clip-path="url(#clip-path)" id={`mt-${i}`} d={pathGenerator(d).slice(0, pathGenerator(d).indexOf('Z'))}   fill={`rgba(${(i + 1) * 15},19,59,${(i + 1) * i * 0.055})`} stroke="black"></path>          
            {/each}
          {/if}
          {#if value === 2}
            {#each kde_theft.features as d, i}
              <path clip-path="url(#clip-path)" id={`t-${i}`} d={pathGenerator(d).slice(0, pathGenerator(d).indexOf('Z'))} fill={`rgba(${(i + 1) * 15},19,59,${(i + 1) * i * 0.055})`} stroke="black"></path>
            {/each}
          {/if}

        </g>

        <g >
          {#each chicagodata.features as d, i}
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          <path d={pathGenerator(d)} fill={"none"} stroke="black" on:click={(event) => {console.log(d)}}></path>
          {/each}
          
        </g>
        {#if value === 1 || value === 3}
        <g >
          {#each chicagodata.features as d, i}
          {@const shape_area_norm = parseInt(d.properties.shape_area) * 0.00000000268936051}
          {@const neighborhood_centroid = d3.polygonCentroid(d.geometry.coordinates[0].map(x => projection(x)))}
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          {#if value === 1 && shape_area_norm > .1 && (d.properties.num_thefts ?? 0)/shape_area_norm > 1 && neighborhood_centroid[0]}
            <text stroke="white" font-size="11px" text-anchor="middle" alignment-baseline="middle" x={neighborhood_centroid[0]} y={neighborhood_centroid[1]}>{d.properties.pri_neigh}</text>
          {/if}  
          
          {#if value === 3 && shape_area_norm > .1 && (d.properties.num_motor_thefts ?? 0)/shape_area_norm > 1 && neighborhood_centroid[0]} 
            <text fill="black" stroke="white" font-size="11px" text-anchor="middle" alignment-baseline="middle" x={neighborhood_centroid[0]} y={neighborhood_centroid[1]}>{d.properties.pri_neigh}</text>
          {/if}          
        {/each}
        </g>
        {/if}          

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
    background-color: blanchedalmond;
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
    transition: background 500ms;
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
      align-items: center;
      flex-direction: column-reverse;
    }
    .sticky {
      top: 25%;
      width: 95%;
			margin: auto;
    }
  }
</style>