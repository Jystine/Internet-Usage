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

    let tooltipPt = null;
    function onPointerMove(event) {
        var index = d3.select(valuemap).attr('Region')
        // const i = d3.bisect(data, countries.invert(d3.pointer(event)[0]))
        // tooltipPt = data[i]
    }

    $: d3.select(svg).selectChild("g")
    .on('mouseover', function (d, i) {
          d3.select(this).transition()
               .duration('50')
               .attr('opacity', '.85'); })
     .on('mouseout', function (d, i) {
          d3.select(this).transition()
               .duration('50')
               .attr('opacity', '1');})

//     var tooltip = d3.select("internet-plot")
//     .append("div")
//     .style("opacity", 0)
//     .attr("class", "tooltip")
//     .style("background-color", "white")
//     .style("border", "solid")
//     .style("border-width", "2px")
//     .style("border-radius", "5px")
//     .style("padding", "5px")
//     var mouseover = function(d) {
//     tooltip
//       .style("opacity", 1)
//     d3.select(this)
//       .style("stroke", "black")
//       .style("opacity", 1)
//     }
//     var mousemove = function(d) {
//     tooltip
//       .html("The exact value of<br>this cell is: " + 10)
//       .style("left", (d3.pointer(event)[0]+70) + "px")
//       .style("top", (d3.pointer(event)[1]) + "px")
//   }
//     var mouseleave = function(d) {
//     d3.selectAll(".Country")
//       .transition()
//       .duration(200)
//       .style("opacity", .8)
//     d3.select(this)
//       .transition()
//       .duration(200)
//       .style("stroke", "transparent")
//   }
//     $: d3.select(svg)
//     .on("mouseover", mouseover)
//     .on("mousemove", mousemove)
//     .on("mouseleave", mouseleave)



    // $: d3.select(svg)
    // .on('mouseover', function(d){
    //   var index = d3.select(this).attr(Object.keys(valuemap));
    //   var percentage = d3.select(this).attr('Percentage');
    //   var region = d3.select(this).attr('Region');
    //   console.log(index)
    //   //var county = index == -1 ? 'unknown' : eduData[index].area_name;
    //   var tooltip = d3.select('#tooltip')
    //   //.style('left', d3.event.pageX + 10 + 'px')
    //   //.style('top', d3.event.pageY + 10 + 'px')
    //   //.style('display', 'block')
    //   .attr('data-education', percentage)
    //   .html(`${region}: ${percentage}`)
    // })
    // .on('mouseout', ()=>d3.select('#tooltip').style('display', 'none'));

    // const colorScale = scaleQuantize([1, 7], schemeBlues[6]);

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
            valuemap[dataYear[i].ISO_num] = {Percentage: dataYear[i].Percentage, Region: dataYear[i].Region};
        }
        return valuemap
    }

    // function colorBin(percentage){
    //     if (percentage >= 0 && percentage < 10) {
    //         return "#f7fbff"
    //     } else if (percentage >= 10 && percentage < 20) {
    //         return "#deebf7"
    //     } else if (percentage >= 20 && percentage < 30) {
    //         return "#c6dbef"
    //     } else if (percentage >= 30 && percentage < 40) {
    //         return "#9ecae1"
    //     } else if (percentage >= 40 && percentage < 50) {
    //         return "#6baed6"
    //     } else if (percentage >= 50 && percentage < 60) {
    //         return "#4292c6"
    //     } else if (percentage >= 60 && percentage < 70) {
    //         return "#2171b5"
    //     } else if (percentage >= 70 && percentage < 80) {
    //         return "#08519c"
    //     } else if (percentage >= 80 && percentage < 90) {
    //         return "#08306b"
    //     } else if (percentage >= 90 && percentage < 100) {
    //         return "#071630"
    //     }
    // }

    
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
     $: console.log(countries);
     $: console.log(valuemap);
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
      overflow = "visible"
    >

    <path d = {path(outline)} fill = "#fff" />
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

    <g class = "legend" stroke = "#000">
        <text x = "1000" y = "70">Percentage Bins</text>
        <circle
        key = 1
        cx = 1000px
        cy = 100px
        fill = "#eff3ff"
        r = "8"
        />
        <text x = "1020" y = "105"> 0 - 20</text>
        <circle
        key = 1
        cx = 1000px
        cy = 130px
        fill = "#bdd7e7"
        r = "8"
        />
        <text x = "1020" y = "135"> 20 - 40</text>
        <circle
        key = 1
        cx = 1000px
        cy = 160px
        fill = "#6baed6"
        r = "8"
        />
        <text x = "1020" y = "165"> 40 - 60</text>
        <circle
        key = 1
        cx = 1000px
        cy = 190px
        fill = "#3182bd"
        r = "8"
        />
        <text x = "1020" y = "195"> 60 - 80</text>
        <circle
        key = 1
        cx = 1000px
        cy = 220px
        fill = "#08519c"
        r = "8"
        />
        <text x = "1020" y = "225"> 80 - 100</text>
        <circle
        key = 1
        cx = 1000px
        cy = 250px
        fill = "#808080"
        r = "8"
        />
        <text x = "1020" y = "255"> No data available</text>
    </g>
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