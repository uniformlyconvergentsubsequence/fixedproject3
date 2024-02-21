<script>
    
    import * as d3 from 'd3';
    import { csvParse, autoType } from 'd3';
    import { onMount } from 'svelte';

    let tempdata = [];
    let chartReady = false
    let sortBy = 'median_age';
    let selectedRegion = null; 
    onMount(async () => {
        const res = await fetch('combined_data.csv');
        const csv = await res.text();
        tempdata = csvParse(csv, autoType);
        console.log(tempdata);
        console.log(tempdata[0])
    let mouse_position = {
        x: 0, y: 0
    };
    let prev_d = null;

    var margin = {top: 10, right: 30, bottom: 50, left: 60},
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
            .domain([10, 50])
            .range([ 0, width ]);
        svg.append("g")
            .attr("transform", "translate(0," + height + ")")
            .call(d3.axisBottom(x))
            .append("text")
            .attr("class", "x-axis-label")
            .attr("x", width / 2)
            .attr("y", margin.bottom)
            .attr("text-anchor", "middle")
            .text("Median Age");
        // Add x axis label
        svg.append("text")
            .attr("class", "x label")
            .attr("text-anchor", "end")
            .attr("x", width / 2)
            .attr("y", height + 35)
            .attr('text-anchor', 'middle')
            .text("Population Median Age");

        // Add Y axis
        var y = d3.scaleLinear()
            .domain([0, 100])
            .range([ height, 0]);
        svg.append("g")
            .call(d3.axisLeft(y))
            .append("text")
            .attr("class", "y-axis-label")
            .attr("transform", "rotate(-90)")
            .attr("x", -height / 2)
            .attr("y", -margin.left + 10)
            .attr("text-anchor", "middle")
            .text("Leader Age");
        // Add Y axis label
        svg.append("text")
            .attr("class", "y label")
            .attr("transform", "rotate(-90)")
            .attr("text-anchor", "middle")
            .attr("x", -height / 2)
            .attr("y", -margin.left + 30)
            .text("Leader Age");

        // Define color scale
        var color = d3.scaleOrdinal(d3.schemeCategory10);

        // Add dots
        svg.append('g')
            .selectAll("dot")
            .data(tempdata)
            .enter()
            .append("circle")
            .attr("cx", function (d) { return x(d.median_age); } )
            .attr("cy", function (d) { return y(d.leader_age); } )
            .attr("r", 5)
            .style("fill", function(d) { return color(d.region); }) 
            .on("mouseover", (event) => {  
                const d = event.target.__data__;
                mouse_position = {
                    x: event.pageX,
                    y: event.pageY
                }
                console.log(d); 
                d3.select(this)
                    .attr("r", 50) 
                    .style("fill", "red") 
                    .style("stroke-width", "2px") 
                    .style("stroke", "black"); 

                // Show tooltip
                d3.select("#tooltip")
                    .style("display", "block")
                    .html(`Country: ${d.Country}<br>Median Age: ${d.median_age}<br>Leader Age: ${d.leader_age}<br>Region: ${d.region}`) // Display the country name and region in the tooltip
                    .style("left", (mouse_position.x+10) + "px")
                    .style("top", (mouse_position.y+10) + "px")
                
            })
            .on("mouseout", function(d) {
                d3.select(this)
                    .attr("r", 5)
                    .style("fill", function(d) { return color(d.region); }) 
                    .style("stroke-width", "1px") 
                    .style("stroke", "none"); 

                // Hide tooltip
                d3.select("#tooltip")
                    .style("display", "none");
            });

        
        // Add legend
        var legend = d3.select("#legend")
            .append("svg")
            .attr("width", 500)
            .attr("height", 300) 
            .selectAll("g")
            .data(color.domain().map(d => d.trim())) 
            .enter()
            .append("g")
            .attr("transform", function(d, i) { return "translate(0," + i * 20 + ")"; })
            .on("click", function(event, d) {
                console.log("Clicked region:", d); 
                console.log("selectedRegion (before):", selectedRegion);

                // If user clicks the same region twice, re-toggle all
                if (selectedRegion === d) {
                        selectedRegion = null;
                        legend.selectAll('text').style('fill', 'black');
                } else {
                        selectedRegion = d;
                        // Toggle class for all legend items based on selectedRegion
                        legend.selectAll('text').style('fill', function(region) {
                            return region === selectedRegion ? 'black' : 'grey';
                        })
                }

                // Update dots
                updateDots(svg);
            });

        legend.append("rect")
            .attr("x", 0)
            .attr("y", 0)
            .attr("width", 18)
            .attr("height", 18)
            .style("fill", color);

        legend.append("text")
            .attr("x", 24)
            .attr("y", 9)
            .attr("dy", ".35em")
            .text(function(d) { return d; });
        
        function updateDots(svg) {
            const sortFunction = sortBy === 'median_age' 
                                 ? (a, b) => d3.ascending(a.leader_age, b.leader_age)
                                 : (a, b) => d3.ascending(a.median_age, b.median_age);

            svg.selectAll('circle') // Re-sort existing dots
               .sort(sortFunction) 
               .transition()
               .duration(500)
               .attr("cx", function(d) { return x(d.median_age); })
               .attr("cy", function(d) { return y(d.leader_age); })
               .style("display", function(d) { 
                
                if (selectedRegion === null) {
                    return "block"; // Show all if no region selected
                } else {
                    return d.region === selectedRegion ? "block" : "none"; 
                }
            }); 

            
        }



    });
</script>

<main>
    <body>
        <h1>Are country leaders older than their population?</h1>
        <h5>Data from 2018</h5>
        <p>Click on legend to filter by region, and hover the data points to see details!</p>
        <div class='row'>
            <div class='column'>
                <div id="my_dataviz"></div>
                <div id="tooltip" style="
                    display: none; 
                    position: absolute; 
                    background-color: white; 
                    padding: 10px; 
                    border: 1px 
                    solid gray;"/>
            </div>
            <div class='column'>
                <div id="legend"></div>
            </div>
        </div>
    </body>
</main>

<style>

    body {
        font-size: 16px;
        color: black;
        font-family:'Franklin Gothic Medium';

    }
    .row {
        display:flex;
    }

    .column {
        flex: 50%;
    }

</style>