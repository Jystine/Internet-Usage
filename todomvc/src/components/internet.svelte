<script>
    import * as d3 from "d3";
    import * as topojson from "topojson-client";
    import { scaleQuantize } from "d3-scale";
    import { schemeBlues } from "d3-scale-chromatic";
    import { onMount } from "svelte";
    //export let data;

    const width = 928;
    const height = 600;
    const marginTop = 20;
    const marginRight = 30;
    const marginBottom = 30;
    const marginLeft = 40;

    let data = [];
    onMount(async () => {
    try {
        const csvData = await d3.csv('https://raw.githubusercontent.com/Jystine/Internet-Usage/main/data/internet_usage.csv');
        data = csvData;
    } catch (error) {
        console.error('Error fetching data:', error);
    }
});
    let svg;
    const projection = d3.geoEqualEarth().fitExtent([[2, marginTop + 2], [width - 2, height]], {type: "Sphere"});
    const path = d3.geoPath(projection);
    let outline = ({type: "Sphere"});
    let land;
    let border;
    let countries = [];
    const colorScale = scaleQuantize([1, 7], schemeBlues[6]);
    let valuemap = new Map(data.map(d => [d.Region, d.Percentage]));
    //const valuemap = d3.map(data, d => d.Region);

    d3.json(
        "https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json"
    ).then((world) => {
        land = topojson.feature(world, world.objects.land);
        border = topojson.mesh(world, world.objects.countries, (a, b) => a !== b)
        countries = topojson.feature(world, world.objects.countries).features;
    });

    $: console.log(data);
    $: console.log(countries);
    $: console.log(valuemap);
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
    <path d = {path(land)} fill = "#000" /><!-- Map this to the percentage values-->
    <!-- <g class = "data"> 
        {#each countries as country} 
​           <path d = {path(land)} fill={colorScale(valuemap.get(country.properties.name))} /> 
​        {/each} 
    </g> -->
    <path d = {path(border)} fill = "none" stroke = "#fff" />
    <path d = {path(outline)} fill = "none" stroke = "#000" />

    </svg>
</div>