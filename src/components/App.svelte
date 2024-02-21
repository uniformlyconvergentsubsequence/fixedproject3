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
    var customOrder = ['Europe', 'Africa', 'South America', 'Central America and the Caribbean', 'North America', 'Middle East', 'Central Asia', "South Asia", 'East and Southeast Asia', 'Australia and Oceania'];

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

        // Make a custom color scale
        var customColors = [
            '#FF0000',
            '#ff7f00',
            "#FFD400",
            '#0095FF',
            '#0040FF',
            '#23628F',
            '#737373',
            '#6B238F',
            '#8F2323',
            "#4F8F23"
        ];
        var color = d3.scaleOrdinal().range(customColors);

        // Add dots
        var dots = svg.append('g')
            .selectAll("dot")
            .data(tempdata)
            .enter()
            .append("circle")
            .attr("cx", function (d) { return x(d.median_age); })
            .attr("cy", function (d) { return y(d.leader_age); })
            .attr("r", 5)
            .style("fill", function (d) { return color(d.region); })
            .on("mouseover", function (event) {
                const d = event.target.__data__;
                mouse_position = {
                    x: event.pageX,
                    y: event.pageY
                };
                console.log(d);
                d3.select(this)
                    .attr("r", 7)
                    .style("fill", "red")
                    .style("stroke-width", "2px")
                    .style("stroke", "black");

                // Show tooltip
                d3.select("#tooltip")
                    .style("display", "block")
                    .html(`Country: ${d.Country}<br>Median Age: ${d.median_age}<br>Leader Age: ${d.leader_age}<br>Region: ${d.region}`) // Display the country name and region in the tooltip
                    .style("left", (mouse_position.x + 10) + "px")
                    .style("top", (mouse_position.y + 10) + "px");
            })
            .on("mouseout", function () {
                d3.select(this)
                    .attr("r", 5)
                    .style("fill", function (d) { return color(d.region); })
                    .style("stroke-width", "1px")
                    .style("stroke", "none");

                // Hide tooltip
                d3.select("#tooltip")
                    .style("display", "none");
            });

        // Function to search for dots by country name dynamically
        document.getElementById("searchBox").addEventListener("input", function () {
            // Whenever the searchBox is typed in, update the variable searchText
            var searchText = this.value.trim().toLowerCase();
            console.log("Search Text: ", searchText);
            
            // Get the matching countries
            var matchingCountries = tempdata.filter(function (d) {
                return searchText && d.Country.toLowerCase().startsWith(searchText);
            });

            // Log the matching countries to the console
            console.log("Matching Countries:", matchingCountries);

            // Iterate over each dot and change opacity based on if the string matches name
            dots.each(function (d) {
                // "match" the country name to the search box content
                var isMatching = searchText && d.Country.toLowerCase().startsWith(searchText);

                // Use an if block to set opacity
                if (searchText) {
                    d3.select(this)
                        .style("opacity", isMatching ? 1 : 0.07);
                } else {
                    d3.select(this).style("opacity", 1);
                }
            });
        });


        

        // Add legend
        var legend = d3.select("#legend")
            .append("svg")
            .attr("width", 500)
            .attr("height", 300) 
            .selectAll("g")
            .data(customOrder, d => d.trim()) 
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
        <div class='row'>
            <div class='left'>
                <div id="my_dataviz"></div>
                <div id="tooltip" style="
                    display: none; 
                    position: absolute; 
                    background-color: white; 
                    padding: 10px; 
                    border: 1px 
                    solid gray;"/>
            </div>
            <div class='right'>
                <label for="searchBox">Search Dot by Name: </label>
                <input type="text" id="searchBox" placeholder="Enter name">
                <br>
                <br>
                <div id="legend"></div>
            </div>
        </div>
        <p>Click on the legend to filter by region and hover the data points to see details!</p>
        
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

    .left {
        flex: 10%;
    }
    .right {
        flex: 90%;
    }
    

</style>