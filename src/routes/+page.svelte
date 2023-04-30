<script lang="ts">
	import { onMount } from 'svelte';
  import chicagodata from './../chicagodata.json';
  import kde_motor_theft from './../kde_motor_theft.json';
  import kde_theft from './../kde_theft.json';
  import chicago_outline from './../chicago_outline.json'
  import * as d3 from 'd3';
  import Scrolly from "./Scrolly.svelte";
  import { fade } from "svelte/transition";
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
      "<p>A closer look at the Loop reveals something sus.</p>",
      "<p>Visualizations of theft and motor theft in Chicago reveal that these crimes are prevalent across the city, with concentrations varying in different neighborhoods. Combating these crimes presents significant challenges for law enforcement and communities alike, including limited resources, complex socio-economic factors, and evolving criminal tactics.<p></p>Additionally, dense urban areas and underreporting of incidents further complicate efforts to address these crimes effectively. Despite these challenges, law enforcement and communities are working together to implement prevention and intervention strategies to reduce the incidence of theft and motor theft in Chicago. By utilizing data-driven approaches, such as the visualizations presented here, we can better understand the scope of the problem and allocate resources more effectively to combat these crimes.</p>"
    ];
    let svg;
    let zoom;
    let audio;
    let tick = 0;

    setInterval(() => {
      tick += 1;
    }, 3000)
    const height = 800;
    const width = 800;
    const viewBox = `0 0 ${height} ${width}`;
    let isZoomed = false;
    $: (value === 5) ? ((isZoomed) ? null : (()=>{isZoomed = true; clicked(null, chicagodata.features[26])})()) :  ((isZoomed) ? (() => {isZoomed = false; reset()})():null);
    let projection: d3.GeoProjection = d3.geoMercator()
    .scale((100 * width))
    .rotate([0, 0])
    .center([-87.717, 41.83724])
    .translate([width/2, height/2]);
    const pathGenerator = d3.geoPath(projection);
    function clicked(event, d) {
      const [[x0, y0], [x1, y1]] = pathGenerator.bounds(d);
      svg.transition().delay(350).duration(750).call(
        zoom.transform,
        d3.zoomIdentity
          .translate(width / 2, height / 2)
          .scale(Math.min(8, 0.9 / Math.max((x1 - x0) / width, (y1 - y0) / height)))
          .translate(-(x0 + x1) / 2, -(y0 + y1) / 2),
      );
      audio.play();
    }

  function reset() {
    svg.transition().delay(350).duration(750).call(
      zoom.transform,
      d3.zoomIdentity,
      d3.zoomTransform(svg.node()).invert([width / 2, height / 2])
    );
  }

    let svgref: any;
    onMount(() => {
      svg = d3.select(svgref);
      function zoomed(event) {
            d3.select("#mapgroup").attr("transform", event.transform);
      }

      zoom = d3
            .zoom()
            .scaleExtent([1, 20])
            .translateExtent([
                [0, 0],
                [height , width],
            ])
            .on("zoom", zoomed);
      svg.call(zoom).on('wheel.zoom', null).on('dblclick.zoom', null);
    })

</script>


<section>
  <div class="hero">
    <h1 >Chicago Data Visualization</h1>
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
    <svg bind:this={svgref} id="mapbox" {viewBox} >
      <g id="mapgroup"> 
        <clipPath id="clip-path">
          <path d={pathGenerator(chicago_outline.features[0])} fill="white" stroke="black" stroke-width="1px"></path>
        </clipPath>
        <rect clip-path="url(#clip-path)" fill="white" height="100%" width="100%"></rect>
        <g id="filled-neighborhoods">
          {#each chicagodata.features as d, i}
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          
            <path d={pathGenerator(d)} fill={value === 0 ? "white" : 
                                                              [3,4,5].includes(value) ? `rgba(244, 15, 151, ${(d.properties.num_motor_thefts ?? 0) / (parseInt(d.properties.shape_area) * 0.00000000268936051 + .1)})` :
                                                              [1,2].includes(value) ? `rgba(244, 15, 151, ${(d.properties.num_thefts ?? 0) / (parseInt(d.properties.shape_area) * 0.00000000268936051 + .1) })` : "none"} stroke="black" on:click={(event) => {console.log(d)}}></path>
          {/each}
        </g>
        <g class="map">
          
          {#if [4].includes(value)}
          <g transition:fade>
            {#each kde_motor_theft.features as d, i}
            <path clip-path="url(#clip-path)" id={`mt-${i}`} d={pathGenerator(d).slice(0, pathGenerator(d).indexOf('Z'))} fill={`rgba(${(i + 1) * 15},19,59,${(i + 1) * i * 0.055})`} stroke="black"></path>          
            {/each}
          </g>
          {/if}
          {#if [2].includes(value)}
          <g transition:fade>
            {#each kde_theft.features as d, i}
              <path clip-path="url(#clip-path)" id={`t-${i}`} d={pathGenerator(d).slice(0, pathGenerator(d).indexOf('Z'))} opacity={value === 2 ? 1 : 0} fill={`rgba(${(i + 1) * 15},19,59,${(i + 1) * i * 0.055})`} stroke="black"></path>
            {/each}
            </g>
          {/if}
        </g>
        {#if [2,4].includes(value)}
        <g id="clickable-neighborhood" transition:fade>
          {#each chicagodata.features as d, i}
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          <path  d={pathGenerator(d)} fill={"none"} stroke="black" on:click={(event) => {clicked(event, d)}}></path>
          {/each}
        </g>
        {/if}
        {#if [1,3].includes(value)}
        <g>
          {#each chicagodata.features as d, i}
          {@const shape_area_norm = parseInt(d.properties.shape_area) * 0.00000000268936051}
          {@const neighborhood_centroid = d3.polygonCentroid(d.geometry.coordinates[0].map(x => projection(x)))}
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          {#if value === 1 && shape_area_norm > .15 && (d.properties.num_thefts ?? 0)/shape_area_norm > 1 && neighborhood_centroid[0]}
            <text transition:fade fill="#efefef"  font-size="11px" text-anchor="middle" alignment-baseline="middle" x={neighborhood_centroid[0]} y={neighborhood_centroid[1]}>{d.properties.pri_neigh}</text>
          {/if}  
          
          {#if value === 3 && shape_area_norm > .15 && (d.properties.num_motor_thefts ?? 0)/shape_area_norm > 1 && neighborhood_centroid[0]} 
            <text transition:fade fill="#efefef"  font-size="11px" text-anchor="middle" alignment-baseline="middle" x={neighborhood_centroid[0]} y={neighborhood_centroid[1]}>{d.properties.pri_neigh}</text>
          {/if}          
        {/each}
        </g>
        {/if}
        {#if value === 5}           
        <g in:fade={{duration: 3000, delay: 1500}} out:fade transform="translate(248, 5)">
          <path d="m307.95364,298.70967l0.102,0.228l3.638,-0.073l0,0.209l-4.031,0.096l-3.351,0.073l-2.842,0.402l-0.046,0.338l-0.001,1.183l-0.082,0.134l-0.026,3.421l-0.03,0.872l-0.01,1.348l-3.84,0.041l-0.032,0.719l-0.207,0.615l-0.175,0.257l-0.479,0.51l-0.473,0.341l-0.7,0.308l-0.632,0.143l-1.108,0.048l-0.458,0.101l-0.287,0.14l-0.339,0.29l-0.271,0.456l-0.102,0.445l0.011,0.587l1.039,-0.006l0.044,1.593l-1.055,0.02l0.027,1.461l0.12,-0.004l0.051,3.664l0.043,0.013l0.088,1.497l0.069,1.886l0.014,1.953l-0.078,2.262l0.053,3.37l-0.113,0.005l0.066,0.474l-0.047,0.296l0.078,4.984l0.49,0.283l0.412,0.388l0.298,0.443l-1.746,0.259l-2.707,0.331l0.931,-1.341l0.361,-0.478l0.208,-0.405l0.184,-0.499l0.138,-0.725l0.013,-0.583l-0.06,-2.352l-0.142,-8.452l-0.132,-7.485l-0.038,-1.414l0.008,-0.508l0.02,-1.315l0.063,-0.475l0.127,-0.455l0.412,-0.839l0.408,-0.52l0.64,-0.539l1.275,-0.775l0.709,-0.58l0.288,-0.332l0.429,-0.665l-4.016,0.021l-2.435,-0.103l-3.23,0.018l-0.299,0.027l-1.913,-0.004l-1.339,-0.069l-0.406,0.005l-0.456,-0.065l0.088,2.798l0.074,4.036l-0.042,1.38l0.044,2.321l0.032,0.454l-0.003,1.211l0.087,4.945l0.016,1.396l0.086,4.295l0.082,4.73l-0.021,0.647l0.052,2.796l-0.048,0.898l-2.105,0.043l-2.485,0.037l-0.06,-2.845l-0.078,-4.863l-0.052,-2.395l-0.056,-1.987l-0.023,-1.296l-0.498,-0.004l-3.78,0.062l0.04,1.733l0.114,5.666l0.102,1.74l0.079,4.263l-3.61,0.076l-0.835,0.027l-1.557,0.009l-0.111,-1.319l-0.208,-1.801l-0.15,-1.801l-0.153,-1.253l-0.363,-2.547l-0.138,-0.781l-0.462,-2.98l-0.274,-1.316l-0.319,-0.99l-0.077,-0.143l-0.668,-0.865l-0.334,-0.552l-0.368,-0.821l-0.138,-0.408l-0.322,-1.162l-0.155,-1.504l-0.128,-0.61l-0.327,-1.196l-0.024,-0.629l-0.119,-0.686l-0.11,-1.118l-0.012,-1.217l0.035,-0.995l0.053,-0.396l-0.012,-0.667l0.13,-2.013l0.08,-0.359l0.229,-1.715l0.05,-0.78l0.168,-1.628l0.151,-0.918l0.091,-0.309l0.207,-0.348l0.236,-0.257l0.596,-0.471l1.213,-1.004l0.249,-0.158l0.248,-0.046l1.088,-0.029l0.802,-0.096l0.686,-0.029l0.562,0.108l1.594,-0.006l1.361,0.047l0.657,0.044l2.095,0.022l1.461,-0.054l0.868,-0.119l0.491,-0.277l1.269,-1.098l0.515,-0.488l0.45,-0.322l0.52,-0.302l0.142,-0.061l0.311,-0.061l0.332,0.016l0.975,0.143l0.243,-0.141l0.35,-0.022l1.248,0.13l2.113,0.338l2.954,0.417l1.498,0.229l1.056,-0.032l0.466,-0.047l1.496,-0.071l0.815,-0.059l2.811,0.034l2.049,-0.114l0.5,-0.089l1.363,0.062l3.957,0.094l3.372,-0.074z" id="svg_1" stroke="#000000" fill="#ff0000"/>
          <ellipse fill="#59c1ea" cx="275.18087" cy="308.59778" id="svg_9" rx="8.21861" ry="4.85829" stroke="#000000"/>
          <ellipse fill="#59c1ea" cx="276.86865" cy="305.54711" id="svg_14" rx="2.2672" ry="0.21474" stroke="#ffffff"/>
          <path fill="#ff0000" stroke="#000000" id="svg_29" d="m261.6183,307.90322c0,-0.10751 -0.10754,-0.10751 -0.10754,-0.10751c0,0 -0.10751,0 -0.10751,0c-0.10754,0 -0.10754,0 -0.10754,0c0,-0.10754 -0.10751,-0.10754 -0.10751,-0.10754c-0.10754,0 -0.10754,0 -0.10754,0c-0.10751,0 -0.10751,0 -0.10751,0c-0.10751,0 -0.21506,0 -0.21506,0c0,0 -0.10751,0 -0.21506,0c0,0 -0.10751,0 -0.10751,0c0,0 -0.10754,0 -0.10754,0c-0.10751,0 -0.10751,0 -0.10751,0c-0.10754,0 -0.10754,0 -0.10754,0c-0.10751,0 -0.10751,0 -0.21503,0c-0.10754,0 -0.13901,0.03149 -0.21506,0.10754c-0.07605,0.07605 -0.1821,0.04932 -0.3226,0.10751c-0.19867,0.08228 -0.22321,-0.04117 -0.32257,0c-0.1405,0.0582 -0.21506,0.10754 -0.32257,0.10754c-0.10754,0 -0.03152,0.03149 -0.10754,0.10751c-0.07602,0.07602 -0.10751,0 -0.21506,0.10754c0,0 -0.10751,0 -0.10751,0c0,0 0,0.10751 -0.10751,0.10751c0,0 0,0.10754 0,0.10754c-0.10754,0.10751 -0.10754,0.10751 -0.21506,0.21503c0,0 0.04114,0.11572 0,0.21506c-0.0582,0.1405 -0.10754,0.21506 -0.10754,0.3226c0,0.10751 -0.10751,0.21506 -0.10751,0.32257c0,0.21506 0,0.32257 0,0.53763c0,0.10754 0,0.32257 0,0.43011c0,0.21506 0,0.32257 0,0.53763c0,0.21506 -0.04941,0.32831 0,0.53763c0.05524,0.23401 0.08279,0.43298 0.10751,0.53766c0.05524,0.23398 0.10754,0.32257 0.10754,0.43008c0,0.10754 0,0.21506 0,0.3226c0,0 0,0.10751 0,0.21506c0,0 0,0.10751 0,0.21503c0,0.21506 0,0.3226 0,0.43011c0,0.10754 0,0.21506 0,0.32257c0,0.10754 0,0.21506 0,0.3226c0,0.10751 0.05228,0.19611 0.10751,0.43011c0.02469,0.10464 0,0.10751 0,0.32257c0,0.10751 0,0.21506 0,0.32257c0,0.10754 0,0.21506 0.10754,0.3226c0,0 0,0.21503 0,0.21503c0,0.10754 0.04932,0.1821 0.10751,0.3226c0.04114,0.09933 0,0.21506 0,0.32257c0,0.21506 0.10751,0.21506 0.10751,0.32257c0,0.21506 0.10754,0.21506 0.10754,0.3226c0,0.10751 0,0.21506 0,0.32257c0,0 0.04932,0.07455 0.10751,0.21506c0.04114,0.09933 0,0.10751 0,0.21506c0,0.10751 -0.07605,0.13901 0,0.21506c0.07605,0.07605 0.10754,0.10751 0.10754,0.10751c0,0 -0.07605,0.03149 0,0.10754c0.07602,0.07602 0.10751,0 0.10751,0.10751c0,0 0.10754,0 0.10754,0c0,0 0.10751,0 0.21506,0.10751c0,0 0,0 0.10751,0c0,0 0.10751,0 0.10751,0c0.10754,0.10754 0.10754,0.10754 0.21506,0.10754c0,0 0.03152,0.03149 0.10754,0.10751c0.07602,0.07602 0.10751,0 0.10751,0c0.10754,0 0.10754,0 0.21506,0c0,0 0,0 0.10751,0c0,0 0.10754,0 0.10754,0c0.10751,0 0.10751,0 0.21506,0c0,0 0.10751,0 0.21506,0c0,0 0.10751,0 0.10751,0c0.10754,0 0.21506,0 0.32257,0c0,0 0.10754,0 0.10754,0c0,0 0.03149,-0.03149 0.10751,-0.10751c0.07605,-0.07605 0.10754,0 0.21506,0c0,0 0,0 0.10754,0c0,0 0,0 0.10751,0c0,0 0,0 0.10754,0c0,0 0,0 0.10751,0c0,0 0.10751,0 0.21506,0l0,0l0.10751,0l0,0"/>
          <path fill="none" stroke="#000000" id="svg_31" d="m276.67206,300.80646c0.10751,0 0.11569,0.04117 0.21506,0c0.1405,-0.0582 0.21506,-0.10754 0.32257,-0.10754c0.10751,0 0.32257,0.10754 0.43011,0.10754c0.21506,0 0.33893,-0.08231 0.53763,0c0.14047,0.0582 0.18207,0.15686 0.32257,0.21506c0.19867,0.08231 0.33078,0.06638 0.43011,0.10751c0.1405,0.0582 0.23141,0.13275 0.43011,0.21506c0.14047,0.0582 0.21503,0.10751 0.32257,0.21506c0.10754,0.10754 0.3226,0.21506 0.43011,0.32257c0.10751,0.10754 0.21503,0.21506 0.32257,0.3226c0.10754,0.10754 0.17389,0.22321 0.21506,0.32257c0.0582,0.1405 0.26437,0.1821 0.32257,0.32257c0.04117,0.09937 0.15686,0.1821 0.21506,0.32257c0.04117,0.09937 0.10754,0.21506 0.10754,0.3226c0,0 0.03146,0.13901 0.10751,0.21506c0.07605,0.07605 0.10754,0.10751 0.10754,0.10751c0,0.10754 0.10751,0.10754 0.10751,0.21506c0,0 0.10754,0.10751 0.10754,0.10751c0,0 0,0.10754 0,0.10754c0,0.10751 0,0.10751 0,0.21506c0,0.10751 0,0.10751 0,0.21506c0,0 0,0.10751 0,0.10751c0,0.10754 0,0.10754 0,0.10754l0,0l0,0.10751"/>
          <path fill="none" stroke="#000000" id="svg_32" d="m261.08066,307.79568c0,0 0,0 0,0.10754l0.10751,0"/>
          <path fill="none" stroke="#000000" id="svg_42" d="m262.0484,318.65591c0,0 0,0 0,-0.10751c0,-0.10754 0.04117,-0.22324 0,-0.3226c-0.0582,-0.1405 -0.10754,-0.21506 -0.10754,-0.32257c0,-0.21506 -0.04932,-0.28961 -0.10751,-0.43011c-0.08228,-0.1987 -0.05228,-0.30362 -0.10751,-0.53763c-0.02472,-0.10468 -0.10754,-0.21506 -0.10754,-0.32257c0,-0.21506 0,-0.3226 0,-0.53766c0,-0.10751 0,-0.21503 0,-0.32257c0,-0.21506 0,-0.32257 0,-0.43011c0,-0.10751 0,-0.21506 0,-0.32257c0,-0.10751 0,-0.21506 0,-0.32257c0,-0.10754 0,-0.21506 0,-0.3226c0,-0.21506 0,-0.32257 0,-0.43008c0,-0.10754 0,-0.10754 0,-0.3226c0,0 0,-0.10751 0,-0.21506c0,-0.10751 0,-0.10751 0,-0.21503c0,-0.10754 0,-0.21506 0,-0.3226c0,-0.10751 0,-0.21506 0,-0.32257c0,-0.10754 0,-0.32257 0,-0.32257c0,-0.21506 0,-0.21506 0,-0.3226c0,-0.10751 0,-0.21506 0,-0.21506c0,-0.10751 0,-0.21506 0,-0.21506c0,-0.10751 0,-0.21503 0,-0.32257c0,-0.10751 0,-0.10751 0,-0.21506c0,-0.10751 0,-0.21506 0,-0.32257c0,0 0.03149,-0.03149 0.10754,-0.10754c0.07602,-0.07602 0,-0.21503 0,-0.21503c0,-0.10754 0,-0.21506 0,-0.21506c0,-0.10754 0,-0.21506 0,-0.3226c0,-0.10751 0.10751,-0.21503 0.10751,-0.21503c0,-0.10754 0,-0.10754 0,-0.21506c0,-0.10754 0,-0.10754 0,-0.10754c0,-0.10751 0,-0.10751 0,-0.10751c0,-0.10754 0,-0.10754 0,-0.21506c0,0 0.10751,-0.10754 0.10751,-0.10754c0,-0.10751 0,-0.10751 0,-0.10751c0,-0.10751 0,-0.10751 0,-0.10751c0,0 0,-0.10754 0,-0.21506c0,0 0,0 0,-0.10754c0,0 0,-0.10751 0,-0.10751c0,0 0,-0.10754 0,-0.10754c0,0 0,-0.10751 0,-0.10751l0,0"/>
        </g>
        {/if}
      </g>
    </svg>
    <audio src="/amongus-round-start.mp3" bind:this={audio}>
    </audio>  </div>
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

  #filled-neighborhoods > path {
    transition: fill 500ms;
  }
  .step-content {
    pointer-events: none;
    font-size: 1rem;
    background: whitesmoke;
    color: #ccc;
    border-radius: 5px;
    padding: .5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 250ms;
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
    pointer-events: none;
    width: 100%;
    margin-top: 1em;
    text-align: center;
    transition: background 250ms;
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