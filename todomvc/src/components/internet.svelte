<script>
    import * as d3 from "d3";
    import * as topojson from "topojson-client";
    import { scaleQuantize } from "d3-scale";
    import { schemeBlues } from "d3-scale-chromatic";
    import { onMount } from "svelte";
    export let data

    const width = 928;
    const height = 600;
    const marginTop = 20;
    const marginRight = 30;
    const marginBottom = 30;
    const marginLeft = 40;

    let svg;
    const projection = d3.geoEqualEarth().fitExtent([[2, marginTop + 2], [width - 2, height]], {type: "Sphere"});
    const path = d3.geoPath(projection);
    let outline = ({type: "Sphere"});
    let land;
    let border;
    let countries = [];
    const colorScale = scaleQuantize([1, 7], schemeBlues[6]);
    let valuemap = {};
    const color = d3
    .scaleSequential()
    .domain([0, 100])
    .interpolator(d3.interpolateBuGn)

    d3.json(
        "https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json"
    ).then((world) => {
        land = topojson.feature(world, world.objects.land);
        border = topojson.mesh(world, world.objects.countries, (a, b) => a !== b)
        countries = topojson.feature(world, world.objects.countries);
    });

    for (let i = 0; i < data.length; i++) {
        console.log(data[0]);
        valuemap[data[i].ISO_num] = {Percentage: data[i].Percentage};
    }

    $: console.log(data);
    $: console.log(countries);
    $: console.log(valuemap);
    $: console.log(land);
</script>

<div class = "internet-plot">
    <svg
      bind:this = {svg}
      {width}
      {height}
      viewBox = "0 0 {width} {height}"
      style = "max-width: 100%; height: auto;"
    >

    <path d = {path(outline)} fill = "#fff" />
    <!-- <path d = {path(land)} fill = "#000" />Map this to the percentage values -->
    <g class = "data"> 
        {#if countries.features !== undefined}
            {#each countries.features as country} 
                {#if (country.id !== undefined && valuemap[Number(country.id)] !== undefined)} 
                    {#if (Number(country.id) in valuemap)}
        ​               <path d = {path(country)} fill={color(valuemap[Number(country.id)].Percentage)} /> 
                    {:else}
                        <path d = {path(country)} fill= "#000" />
                    {/if}
                {/if}
    ​        {/each} 
        {/if}
    </g>
    <path d = {path(border)} fill = "none" stroke = "#fff" />
    <path d = {path(outline)} fill = "none" stroke = "#000" />

    </svg>
</div>