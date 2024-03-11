<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    
    export let margin = { top: 60, right: 60, bottom: 60, left: 60 };
    export let width = 600 - margin.left - margin.right;
    export let height = 400 - margin.top - margin.bottom;

    let xAxisColumn = 'StudyHoursperWeek'; // Default x-axis column
    let yAxisColumn = 'AverageGradeReceived'; // Default y-axis column
    let selectedCourse = 'All Courses'; // Default selected course

    

    // Function to create the chart
    function createChart(data, course) {
        selectedCourse = course;
        
        d3.select("#my_dataviz").select("svg").remove();
        d3.select("#my_dataviz").select(".tooltip").remove();

        // Append the SVG object to the div with id "my_dataviz"
        let svg = d3.select("#my_dataviz")
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

        // Filters out all the Nan values and course
        let filteredData;

        if (selectedCourse === "All Courses") {
            filteredData = data.filter(d=> d.selectedYAxis !== "");
        } else {
            filteredData = data.filter(d => d.CourseID === selectedCourse);
            filteredData = filteredData.filter(d=> d.selectedYAxis !== "")
        }

        // Add X axis
        var x = d3.scaleLinear()
            .domain([0, 20])
            .range([ 0, width ]);
        svg.append("g")
            .attr("transform", "translate(0," + height + ")")
            .call(d3.axisBottom(x));

        // Add Y axis
        var y = d3.scaleLinear()
            .domain([0, 4])
            .range([ height, 0]);
        svg.append("g")
            .call(d3.axisLeft(y));
        
        // Tooltip
        var tooltip = d3.select("#my_dataviz")
            .append("div")
            .style("opacity", 0)
            .attr("class", "tooltip")
            .style("border", "solid")
            .style("border-width", "2px")
            .style("border-radius", "5px")
            .style("padding", "10px") 
            .style("text-align", "center") // Center the text inside the tooltip

        // Mouseover function
        var mouseover = function(d) {
            tooltip.style("opacity", 1)
            d3.selectAll(".dot")
                .transition()
                .duration(30)
                .style("fill", "lightgrey")
                .attr("r", 100)
        }

        // Mousemove function
        var mousemove = function(event, d) {
            const containerRect = svg.node().getBoundingClientRect();
            const offsetX = event.pageX - containerRect.left;
            const offsetY = event.pageY - containerRect.top;

            // Enhanced Tooltip HTML with inline CSS for styling
            tooltip.html(`
                <div style="font-size: 20px; font-weight: bold; margin-bottom: 5px;">${d.CourseID}</div>
                <div style="font-size: 16px;">
                    <p style="margin: 2px 0;"><span style="font-weight: bold;">Quarter:</span> ${d.Quarter}</p>
                    <p style="margin: 2px 0;"><span style="font-weight: bold;">Instructor:</span> ${d.Instructor}</p>
                    <p style="margin: 2px 0;"><span style="font-weight: bold;">Average Study Hour per Week:</span> ${d[xAxisColumn]} hours</p>
                    <p style="margin: 2px 0;"><span style="font-weight: bold;">Average Grade Received:</span> ${d[yAxisColumn]}</p>
                </div>
            `)
            .transition()
            .duration(200)
            .style("opacity", .9); // Smooth transition for appearance

            // Position adjustments similar to previous example
            const tooltipRect = tooltip.node().getBoundingClientRect();
            let left = offsetX + 20; // Default right offset
            let top = offsetY + 20; // Default top offset

            // Adjust if tooltip goes beyond the SVG container
            if (containerRect.right - offsetX < tooltipRect.width + 40) {
                left = offsetX - tooltipRect.width - 20;
            }
            if (containerRect.bottom - offsetY < tooltipRect.height + 40) {
                top = offsetY - tooltipRect.height - 20;
            }

            tooltip.style("left", left + "px")
                .style("top", top + "px")
                .style("background-color", "#f9f9f9") // Background color
                .style("box-shadow", "0px 0px 10px rgba(0, 0, 0, 0.5)"); // Shadow for depth
        }


        // Mouseleave function
        var mouseleave = function(d) {
            tooltip.transition()            
            .duration(200)
            .style("opacity", 0)
            
        }

        // X-axis label
        svg.append("text")
            .attr("text-anchor", "end")
            .attr("x", width/2 + margin.left + 20)
            .attr("y", height + margin.top - 20)
            .text('Study Hours per Week');

        // Y-axis label
        svg.append("text")
            .attr("text-anchor", "end")
            .attr("transform", "rotate(-90)")
            .attr("y", -margin.left + 20)
            .attr("x", -margin.top -  height/2 + 120)
            .text('Average Grades Received')

        
        // Dots
        svg.append('g')
            .selectAll("dot")
            .data(filteredData)
            .enter()
            .append("circle")
            .attr("cx", function (d) { return x(parseFloat(d[xAxisColumn])); } )
            .attr("cy", function (d) { return y(parseFloat(d[yAxisColumn])); } )
            .attr("r", 5)
            .style("fill", function(d) { 
                let courseNumber = parseInt(d.Course.replace("DSC", "").trim());
                if (courseNumber >= 100) {
                    return "blue"; // Color for upper division courses
                } else {
                    return "red"; // Color for lower division courses
                }
            })
            .style("opacity", 0.3)
            .style("stroke", "white")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave)

        // Legend
        var legend = svg.append("g")
            .attr("class", "legend")
            .attr("transform", "translate(" + (width - 100) + "," + (height - 100) + ")");

        // Upper division course color indicator
        legend.append("rect")
            .attr("x", 10)
            .attr("y", 10)
            .attr("width", 20)
            .attr("height", 20)
            .style("fill", "blue");

        legend.append("text")
            .attr("x", 40)
            .attr("y", 25)
            .text("Upper Division");

        // Lower division course color indicator
        legend.append("rect")
            .attr("x", 10)
            .attr("y", 40)
            .attr("width", 20)
            .attr("height", 20)
            .style("fill", "red");

        legend.append("text")
            .attr("x", 40)
            .attr("y", 55)
            .text("Lower Division");

    }

    // Use onMount to execute code after component is mounted in the browser
    onMount(() => {
        fetch('./capes_dsc_clean_v1.csv')
            .then(response => response.text())
            .then(csv => {
                let data = d3.csvParse(csv);
                //Button with options
                let allCourses = new Set(data.map(d => d.CourseID));
                allCourses.add("All Courses")

                // Initialize the button
                const dropdownButton = d3.select("#dropdownContainer")
                .append('select')
                .on("change", function() {
                        createChart(data, this.value);
                    });

                // add the options to the button
                dropdownButton // Add a button
                    .selectAll('option')
                    .data(Array.from(allCourses))
                    .enter()
                    .append('option')
                    .text(d => d)
                    .attr("value", d => d);
                
                
                createChart(data, selectedCourse);

            })

            
            .catch(error => {
                console.error('Error loading data:', error);
            });
    });

</script>



<div id="my_dataviz" style="position: relative;">
    <div id="dropdownContainer" style="position: absolute; top: 0px; right: 20px;">
    </div>
</div>
