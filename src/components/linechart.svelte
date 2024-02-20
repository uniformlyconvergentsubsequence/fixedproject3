<script>
  import * as d3 from 'd3';

  // set the dimensions and margins of the graph
  var margin = { top: 10, right: 30, bottom: 30, left: 60 },
    width = 460 - margin.left - margin.right,
    height = 400 - margin.top - margin.bottom;

  let svg; // To store the SVG reference

  d3.csv("combined_data.csv", function (data) {
    // Add X axis
    var x = d3.scaleLinear()
      .domain([0, 10])
      .range([0, width]);

    // Add Y axis
    var y = d3.scaleLinear()
      .domain([0, 10])
      .range([height, 0]);

    // Color scale 
    var color = d3.scaleOrdinal()
      .domain(["setosa", "versicolor", "virginica"])
      .range(["#440154ff", "#21908dff", "#fde725ff"]);

    // Convert data to pie layout
    var pie = d3.pie()
      .value(function(d) { return d.value; });

    // Generate arc paths
    var arc = d3.arc()
      .innerRadius(0)
      .outerRadius(Math.min(width, height) / 2);

    // Generate pie chart
    svg = d3.select('div')
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', 'translate(' + (width / 2 + margin.left) + ',' + (height / 2 + margin.top) + ')')
      .selectAll('path')
      .data(pie(data))
      .enter()
      .append('path')
      .attr('d', arc)
      .attr('fill', function(d) { return color(d.data.Species); });
  });

</script>

<div bind:this={svg} width={width + margin.left + margin.right} height={height + margin.top + margin.bottom}>
</div>