<script>
    let placeholder = "What do you need to do?";
    let todo_text = "";
    import * as d3 from 'd3';
    
    let tempdata = [];
    onMount(async () => {
        const res = await fetch('combined_data.csv');
        const csv = await res.text();
        tempdata = d3.csvParse(csv, d3.autoType);
        console.log(tempdata);
    });
</script>
