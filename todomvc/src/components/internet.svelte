<script>
    import * as d3 from "d3";
    import * as topojson from "topojson-client";
    import { scaleQuantize } from "d3-scale";
    import { schemeBlues } from "d3-scale-chromatic";
    import RangeSlider from "svelte-range-slider-pips";
    export let data

    const width = 928;
    const height = 600;
    const marginTop = 20;
    const marginRight = 30;
    const marginBottom = 30;
    const marginLeft = 40;

    let svg;
    const projection = d3.geoEquirectangular().fitExtent([[2, marginTop + 2], [width - 2, height]], {type: "Sphere"});
    const path = d3.geoPath(projection);
    let outline = ({type: "Sphere"});
    let land;
    let border;
    let countries = [];
    const colorScale = scaleQuantize([1, 7], schemeBlues[6]);

    const years = ['2000', '2005', '2010', '2015', '2020'];
    $: valuemap = getYear(data, chosenYear);
    let slider_label = 'Year';
    let chosenYear = '4';
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

    function getYear(data, chosenYear) {
        let valuemap = {}
        let dataYear = data.filter(function(d) {
            return d.Year == years[chosenYear]
        });
        for (let i = 0; i < dataYear.length; i++) {
            valuemap[dataYear[i].ISO_num] = {Percentage: dataYear[i].Percentage};
        }
        return valuemap
    }

    function colorBin(percentage){
        if (percentage >= 0 && percentage < 10) {
            return "#f7fbff"
        } else if (percentage >= 10 && percentage < 20) {
            return "#deebf7"
        } else if (percentage >= 20 && percentage < 30) {
            return "#c6dbef"
        } else if (percentage >= 30 && percentage < 40) {
            return "#9ecae1"
        } else if (percentage >= 40 && percentage < 50) {
            return "#6baed6"
        } else if (percentage >= 50 && percentage < 60) {
            return "#4292c6"
        } else if (percentage >= 60 && percentage < 70) {
            return "#2171b5"
        } else if (percentage >= 70 && percentage < 80) {
            return "#08519c"
        } else if (percentage >= 80 && percentage < 90) {
            return "#08306b"
        } else if (percentage >= 90 && percentage < 100) {
            return "#071630"
        }
    }

    
    function colorBin2(percentage){
        if (percentage >= 0 && percentage < 20) {
            return "#eff3ff"
        } else if (percentage >= 20 && percentage < 40) {
            return "#bdd7e7"
        } else if (percentage >= 40 && percentage < 60) {
            return "#6baed6"
        } else if (percentage >= 60 && percentage < 80) {
            return "#3182bd"
        } else if (percentage >= 80 && percentage < 100) {
            return "#08519c"
        }
    }


    // $: console.log(filterYear("2021"))
 
    // $: console.log(data);
    // $: console.log(countries);
    // $: console.log(valuemap);
    // $: console.log(dataYear);
    $: console.log(years[chosenYear]);
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
                {#if (country.id !== undefined || valuemap[Number(country.id)] !== undefined)} 
                    {#if (Number(country.id) in valuemap)}
        ​               <path d = {path(country)} stroke = "#000" fill={colorBin2(valuemap[Number(country.id)].Percentage)} /> 
                {:else}
                    <path d = {path(country)} stroke = "#000" fill= "#808080" />
                    {/if}
                {/if}
    ​        {/each} 
        {/if}
    </g>
    <path d = {path(border)} fill = "none" stroke = "#000" />
    <path d = {path(outline)} fill = "none" stroke = "#000" />
    <!-- <div class = "legend">
        svg.append("circle").attr("cx",200).attr("cy",130).attr("r", 6).style("fill", "#69b3a2")
    </div> -->
    </svg>
</div>

<div class = "slider">
    <label>{slider_label}</label>
    <RangeSlider 
    formatter = {v => years[v]}
    min = 0 
    max = 4
    float = true 
    all = 'label' 
    ariaLabels = {["2000", '2005', '2010', '2015', '2020']}
    on:change = {(e) => {
        chosenYear = e.detail.value
    }}
    pips = true />

</div>

<div class = "legend">
    <g stroke = "#000">
        <circle
        key = 1
        cx = 1000px
        cy = 100px
        fill = "000"
        r = "8"
        />
    </g>
</div>

<!-- <div class = "overlay">
    <label>{slider_label}</label>
    <input
        id = "slider"
        type = "range"
        min = "0"
        max = "6"
        list = "steplist"
        bind:value = {chosenYear}
    />
    <datalist id = "steplist">
        <option value = 0 class = "sliderlabel" label = "2000"></option>
        <option value = 1 class = "sliderlabel" label = "2005"></option>
        <option value = 2 class = "sliderlabel" label = "2010"></option>
        <option value = 3 class = "sliderlabel" label = "2015"></option>
        <option value = 4 class = "sliderlabel" label = "2019"></option>
        <option value = 5 class = "sliderlabel" label = "2020"></option>
        <option value = 6 class = "sliderlabel" label = "2021"></option>
    </datalist>
</div> !-->

<style>
    .slider {
        margin: auto;
        top: 50%;
        transform: translate(0, -50%);
        width: 50%;
    }
</style>