<script>
import {
    onMount
} from 'svelte';
import * as d3 from 'd3';
import Svelecte from 'svelecte';

let rawData = [];
let departments = [];
let numericalColumns = [{
        value: 'StudyHoursPerWeek',
        label: 'Study Hours Per Week'
    },
    {
        value: 'AverageGradeReceived',
        label: 'Average Grade Received'
    },
    {
        value: 'PercentageRecommendedClass',
        label: 'Percentage Recommended Class'
    }
];
let selectedDepartment = null;
let selectedColumn = null;

// Fetch and process your data
async function fetchData() {
    const response = await fetch('capes_grouped.csv');
    const text = await response.text();
    rawData = d3.csvParse(text);
    const departmentSet = new Set(data.map(item => item.dept));
    departments = Array.from(departmentSet).map(dept => ({ value: dept, label: dept })).sort((a, b) => a.label.localeCompare(b.label));
    departments = Array.from(new Set(rawData.map(d => ({
        value: d.dept,
        label: d.dept
    })))).sort((a, b) => a.label.localeCompare(b.label));
}

// Draw histogram
function drawHistogram() {
    // Ensure selections are made
    if (!selectedColumn || !selectedDepartment) return;

    const filteredData = rawData.filter(d => d.dept === selectedDepartment.value);
    // Histogram rendering logic here, using `filteredData` and `selectedColumn.value`
    const svg = d3.select("#histogram");
    svg.selectAll("*").remove(); // Clear existing visualization

    const margin = {
            top: 20,
            right: 20,
            bottom: 30,
            left: 40
        },
        width = 450 - margin.left - margin.right,
        height = 400 - margin.top - margin.bottom;
    svg.attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom);

    const g = svg.append("g").attr("transform", `translate(${margin.left},${margin.top})`);

    // Define scales
    const x = d3.scaleLinear()
        .domain([0, d3.max(filteredData, d => +d[selectedColumn.value])])
        .range([0, width]);

    // Generate bins
    const histogram = d3.histogram()
        .value(d => d[selectedColumn.value])
        .domain(x.domain())
        .thresholds(x.ticks(40));

    const bins = histogram(filteredData);

    // Y scale
    const y = d3.scaleLinear()
        .domain([0, d3.max(bins, d => d.length)])
        .range([height, 0]);

    // Draw bars
    g.selectAll("rect")
        .data(bins)
        .enter().append("rect")
        .attr("x", d => x(d.x0) + 1)
        .attr("transform", d => `translate(${x(d.x0)},${y(d.length)})`)
        .attr("width", d => x(d.x1) - x(d.x0) - 1)
        .attr("height", d => height - y(d.length))
        .style("fill", "#69b3a2");

    // Add axes
    g.append("g").call(d3.axisLeft(y));
    g.append("g").attr("transform", `translate(0,${height})`).call(d3.axisBottom(x));

}

onMount(() => {
    fetchData();
});

$: if (selectedDepartment && selectedColumn) {
    drawHistogram();
}
</script>

<div>
    <Svelecte bind:value={selectedDepartment} options={departments} placeholder="Select a Department" />
    <Svelecte bind:value={selectedColumn} options={numericalColumns} placeholder="Select a Numerical Column" />
</div>
<svg id="histogram"></svg>
