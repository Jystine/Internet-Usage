<script>
    import * as d3 from "d3";
    import * as topojson from "topojson-client";
    export let data;

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

    d3.json(
        "https://cdn.jsdelivr.net/npm/world-atlas@2/countries-50m.json"
    ).then((world) => {
        land = topojson.feature(world, world.objects.land);
        border = topojson.mesh(world, world.objects.countries, (a, b) => a !== b)
    });

    $: console.log(data);
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
    <path d = {path(land)} fill = "#000" /> <!-- Map this to the percentage values-->
    <path d = {path(border)} fill = "none" stroke = "#fff" />
    <path d = {path(outline)} fill = "none" stroke = "#000" />

    </svg>
</div>