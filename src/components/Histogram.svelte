<script>
import {
    onMount
} from 'svelte';
import * as d3 from 'd3';

let allCourses = [];
let departments = [];
let selectedDepartment = '';
let selectedCourseNum = '';
let uniqueCourses = [];
const numericalColumns = ['StudyHoursperWeek', 'AverageGradeReceived', 'PercentageRecommendedClass', 'TotalEnrolledinCourse'];

let svgs = []; // Array to hold svg elements for each histogram
const margin = {
    top: 10,
    right: 30,
    bottom: 90,
    left: 40
}; // Increased bottom margin for labels
const width = 600 - margin.left - margin.right; // Adjusted width for side-by-side layout
const height = 300 - margin.top - margin.bottom;

async function loadData() {
    const response = await fetch('capes.csv');
    const text = await response.text();
    const data = d3.csvParse(text);
    allCourses = data.map(d => ({
        ...d,
        TotalEnrolledinCourse: +d.TotalEnrolledinCourse,
        StudyHoursperWeek: +d.StudyHoursperWeek,
        AverageGradeReceived: +d.AverageGradeReceived,
        PercentageRecommendedClass: +d.PercentageRecommendedClass
    }));
    // Extract unique departments
    departments = Array.from(new Set(allCourses.map(d => d.dept)));
}

function initializeBlankHistograms() {
    svgs.forEach((svg, index) => {
        // Setup scales
        const x = d3.scaleLinear()
            .domain([0, 100]) // Assuming a generic range; adjust as needed
            .range([0, width]);
        const y = d3.scaleLinear()
            .domain([0, 10]) // Assuming a generic range; adjust as needed
            .range([height, 0]);

        // Clear previous
        svg.selectAll("*").remove();

        // Draw axes
        svg.append("g")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(x));

        svg.append("g")
            .call(d3.axisLeft(y));
    });
}

onMount(() => {
    loadData();
    // Initialize SVGs for each numerical column
    numericalColumns.forEach((col, index) => {
        svgs[index] = d3.select(`#histogram${index}`)
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);
    });
    initializeBlankHistograms();
});

function updateUniqueCourses() {
    const filteredCourses = allCourses.filter(course => course.dept === selectedDepartment);
    uniqueCourses = filteredCourses.reduce((acc, current) => {
        const x = acc.find(item => item.CourseNum === current.CourseNum);
        if (!x) {
            return acc.concat([current]);
        } else {
            return acc;
        }
    }, []);
    selectedCourseNum = ''; // Reset selected course
}

function renderHistograms() {
    if (!selectedCourseNum) {
        initializeBlankHistograms(); // Re-initialize blank histograms if no course is selected
        return;
    }

    numericalColumns.forEach((col, index) => {
        const svg = svgs[index];
        const data = allCourses.filter(course => course.CourseNum === selectedCourseNum).map(d => +d[col]);

        if (data.length === 0) return; // Check if data is empty

        svg.selectAll("*").remove(); // Clear previous histogram

        // Adding X-axis title
        // Custom thresholds for specific histograms
        let thresholds;
        if (col === 'AverageGradeReceived') {
            thresholds = d3.range(0, 4.3, .33333); // Adjusted example range for grades
        } else if (col === 'PercentageRecommendedClass') {
            thresholds = d3.range(0, 105, 10); // Adjusted example range for percentages
        } else if (col === 'StudyHoursperWeek') {
            thresholds = d3.range(0, d3.max(data) + 1, 2); // Generic range for other columns
        } else {
            thresholds = d3.range(0, d3.max(data) + 1, 20)
        }

        const x = d3.scaleLinear()
            .domain([0, d3.max(data) + d3.max(data) / 4])
            .range([0, width]);

        const histogram = d3.histogram()
            .value(d => d)
            .domain(x.domain())
            .thresholds(thresholds);

        const bins = histogram(data);

        const y = d3.scaleLinear()
            .domain([0, d3.max(bins, d => d.length)])
            .range([height, 0]);

        svg.append("g")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(x))
            .selectAll("text")
            .attr("transform", "translate(-10,0)rotate(-45)")
            .style("text-anchor", "end");

        svg.append("text")
            .attr("text-anchor", "middle")
            .attr("x", width / 2 + margin.left)
            .attr("y", height + margin.top + 40) // Adjust this value to move the title further down from the X-axis
            .style("font-size", "12px")
            .text(col);


        // Adding Y-axis title
        svg.append("text")
            .attr("text-anchor", "middle")
            .attr("transform", "rotate(-90)") // Rotate the text for vertical orientation
            .attr("y", -margin.left +10) // Adjust these values to position the Y-axis title correctly
            .attr("x", -height / 2 + margin.top -30)
            .style("font-size", "12px")
            .text("Frequency");




        svg.append("g")
            .call(d3.axisLeft(y));

        svg.selectAll("rect")
            .data(bins)
            .enter()
            .append("rect")
            .attr("x", 1)
            .attr("transform", d => `translate(${x(d.x0)},${y(d.length)})`)
            .attr("width", d => Math.max(0, x(d.x1) - x(d.x0) - 1))
            .attr("height", d => height - y(d.length))
            .style("fill", "#69b3a2");
    });
}

$: if (selectedDepartment) {
    updateUniqueCourses();
}
$: selectedCourseNum, renderHistograms();
</script>

<style>
.dashboard-container {
    background-color: white;
    width: 1200px;
    padding: 20px;
}

.dropdown-container {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
}

select {
    width: 48%;
    padding: 10px;
    font-size: 16px;
    cursor: pointer;
}

.histograms-container {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
}

.histogram-container {
    display: flex;
    justify-content: center;
    align-items: center;
}

svg {
    width: 100%;
    height: auto;
    background-color: #f9f9f9;
    border: 1px solid #ccc;
}

@media (max-width: 768px) {
    .histograms-container {
        grid-template-columns: 1fr;
        /* Stack the histograms in a single column on smaller screens */
    }

    svg {
        width: 100%;
        /* Make SVGs full width on smaller screens */
    }
}
</style>

<div class="dashboard-container">
    <div class="dropdown-container">
        <select bind:value={selectedDepartment}>
            <option value="">Select a Department</option>
            {#each departments as department}
            <option value={department}>{department}</option>
            {/each}
        </select>

        <select bind:value={selectedCourseNum} disabled={!selectedDepartment}>
            <option value="">Select a Course</option>
            {#each uniqueCourses as course}
            <option value={course.CourseNum}>{course.CourseTitle || 'Course'}</option>
            {/each}
        </select>
    </div>

    <div class="histograms-container">
        <div class="histogram-container" id="histogram0">
        </div>
        <div class="histogram-container" id="histogram1">
        </div>
        <div class="histogram-container" id="histogram2">
        </div>
        <div class="histogram-container" id="histogram3">
        </div>
    </div>
</div>
