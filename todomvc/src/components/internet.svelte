<script>
    import * as d3 from "d3";
    import * as topojson from "topojson-client";
    import RangeSlider from "svelte-range-slider-pips";
    export let data

    const width = 928;
    const height = 500;
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

    // let tooltipPt = null;
    // function onPointerMove(event) {
    //     var index = d3.select(valuemap).attr('Region')
    //     // const i = d3.bisect(data, countries.invert(d3.pointer(event)[0]))
    //     // tooltipPt = data[i]
    // }

    // $: d3.select(svg).selectChild("g")
    // .on('mouseover', function (d, i) {
    //       d3.select(this).transition()
    //            .duration('50')
    //            .attr('opacity', '.85'); })
    //  .on('mouseout', function (d, i) {
    //       d3.select(this).transition()
    //            .duration('50')
    //            .attr('opacity', '1');})

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
    let filterPercentage = [];
    let everFiltered = false;
    let minMax = [0, 100];
    $: valuemap = getYear(data, chosenYear);
    $: subset = filterBins(valuemap, minMax[0], minMax[1])
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

    function trackPercent(filterPercentage, min, max){
        if (filterPercentage.includes(min) === false && filterPercentage.includes(max) === false) {
            filterPercentage.push(min)
            filterPercentage.push(max)
        } else if (filterPercentage.includes(min) && filterPercentage.includes(max)) {
            delete filterPercentage[filterPercentage.indexOf(min)]
            delete filterPercentage[filterPercentage.indexOf(max)]
        }
        return filterPercentage
    }

    function filterBins(valuemap, min = 0, max = 100) {
        let subset = {}
        for (const key in valuemap) {
            if (valuemap[key].Percentage >= min && valuemap[key].Percentage < max) {
                subset[key] = {Percentage: valuemap[key].Percentage, Region: valuemap[key].Region};
            }
        }
        return subset
    }

    let resetColor = "#000"

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
                {#if (country.id !== undefined || subset[Number(country.id)] !== undefined)} 
                    {#if (Number(country.id) in subset)}
        ​               <path d = {path(country)} stroke = "#000" fill={colorBin2(subset[Number(country.id)].Percentage)} /> 
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
        <text x = "990" y = "70" style = "font-size: 22" font-weight = bold>Percentage Bins</text>
        <circle
        key = 1
        cx = 1000px
        cy = 100px
        fill = "#eff3ff"
        r = "15"
        on:click = {() => {subset = filterBins(valuemap, 0, 20), minMax = [0, 20]}}
        on:mouseenter = {() => {
        }}
        />
        <text x = "1030" y = "107" style = "font-size: 22"> 0 - 20</text>
        <circle
        key = 1
        cx = 1000px
        cy = 140px
        fill = "#bdd7e7"
        r = "15"
        on:click = {() => {subset = filterBins(valuemap, 20, 40), minMax = [20, 40]}}
        style = "hover = cursor:pointer"
        />
        <text x = "1030" y = "147" style = "font-size: 22"> 20 - 40</text>
        <circle
        key = 1
        cx = 1000px
        cy = 180px
        fill = "#6baed6"
        r = "15"
        on:click = {() => {subset = filterBins(valuemap, 40, 60), minMax = [40, 60]}}
        />
        <text x = "1030" y = "187" style = "font-size: 22"> 40 - 60</text>
        <circle
        key = 1
        cx = 1000px
        cy = 220px
        fill = "#3182bd"
        r = "15"
        on:click = {() => {subset = filterBins(valuemap, 60, 80), minMax = [60, 80]}}
        />
        <text x = "1030" y = "227" style = "font-size: 22"> 60 - 80</text>
        <circle
        key = 1
        cx = 1000px
        cy = 260px
        fill = "#08519c"
        r = "15"
        on:click = {() => {subset = filterBins(valuemap, 80, 100), minMax = [80, 100]}}
        />
        <text x = "1030" y = "267" style = "font-size: 22"> 80 - 100</text>
        <circle
        key = 2
        cx = 1000px
        cy = 300px
        fill = "#808080"
        r = "15"
        />
        <text x = "1030" y = "307" style = "font-size: 22"> No data available</text>
        <text 
        key = 1
        x = "1030" 
        y = "347" 
        fill = {resetColor}
        style = "font-size: 22"
        on:mouseover = {function(e) {
            resetColor = "white"
        }}
        on:mouseout = {function(e) {
            resetColor = "#000"
        }}
        on:click = {() => {subset = filterBins(valuemap, 0, 100), minMax = [0, 100]}} 
        > Reset Filter</text>
    </g>
    </svg>
</div>

<div class = "slider">
    <label size = "22">{slider_label}</label>
    <RangeSlider 
    formatter = {v => years[v]}
    min = 0 
    max = 4
    float = true 
    all = 'label' 
    ariaLabels = {["2000", '2005', '2010', '2015', '2020']}
    on:change = {(e) => {
        chosenYear = e.detail.value
        valuemap = getYear(data, chosenYear)
        subset = filterBins(valuemap, minMax[0], minMax[1])
    }}
    pips = true />

</div>

<style>
    .slider {
        margin: auto;
        top: 50%;
        transform: translate(0, 30%);
        width: 50%;
    }
    .slider label {
        font-size: 20px
    }
    .legend circle[key = "1"]:hover{
        cursor: pointer;
        opacity: 0.60;
    }
    .legend text[key = "1"]:hover{
        cursor: pointer;
    }
</style>