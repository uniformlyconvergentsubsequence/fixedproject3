<!-- Create a div where the graph will take place -->
<div id="my_dataviz"></div>
<script>
    
    import * as d3 from 'd3';
    import { csvParse, autoType } from 'd3';
    import { onMount } from 'svelte';

    let tempdata = [];
    let chartReady = false
    onMount(async () => {
        const res = await fetch('combined_data.csv');
        const csv = await res.text();
        tempdata = csvParse(csv, autoType);
        console.log(tempdata);
        chartReady = true;
    
    var margin = {top: 10, right: 30, bottom: 30, left: 60},
            width = 460 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;

        // append the svg object to the body of the page
        var svg = d3.select("#my_dataviz")
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform",
                "translate(" + margin.left + "," + margin.top + ")");

        // Add X axis
        var x = d3.scaleLinear()
            .domain([0, 10])
            .range([ 0, width ]);
        svg.append("g")
            .attr("transform", "translate(0," + height + ")")
            .call(d3.axisBottom(x));

        // Add Y axis
        var y = d3.scaleLinear()
            .domain([0, 100])
            .range([ height, 0]);
        svg.append("g")
            .call(d3.axisLeft(y));

        // Add dots
        svg.append('g')
            .selectAll("dot")
            .data(tempdata)
            .enter()
            .append("circle")
            .attr("cx", function (d) { return x(d.years); } )
            .attr("cy", function (d) { return y(d.age); } )
            .attr("r", 1.5)
            .style("fill", "#69b3a2");
    });
    
</script>
