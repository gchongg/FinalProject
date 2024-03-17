<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    // set the dimensions and margins of the graph
    export let margin = { top: 60, right: 60, bottom: 60, left: 60 };
    export let width = 700 - margin.left - margin.right;
    export let height = 500 - margin.top - margin.bottom;

    let selectedCourse = 'All Courses';
        
    function createBubble(data) {
        d3.select("#bubble_plot").select("svg").remove();

        let svg = d3.select("#bubble_plot")
            .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
            .append("g")
                .attr("transform",
                    "translate(" + margin.left + "," + margin.top + ")");

        // Aggregate data by CourseID
        data = data.filter(d=> d.AverageGradeReceived !== "");
        var groupedData = d3.group(data, d => d.CourseID);
        var aggregatedData = Array.from(groupedData, ([key, values]) => {
            return {
                CourseID: key,
                AverageGradeReceived: d3.mean(values, d => parseFloat(d.AverageGradeReceived)),
                TotalEnrolledinCourse: d3.mean(values, d => parseFloat(d.TotalEnrolledinCourse)),
                StudyHoursperWeek: d3.mean(values, d => parseFloat(d.StudyHoursperWeek))
            };
        });

        let filteredData;
        filteredData = aggregatedData;

        //console.log(filteredData);
        // Add X axis
        var x = d3.scaleLinear()
            .domain([0, d3.max(filteredData, d => parseFloat(d['TotalEnrolledinCourse']))])
            .range([ 0, width ]);
        svg.append("g")
            .attr("transform", "translate(0," + height + ")")
            .call(d3.axisBottom(x));

        // Add Y axis
        var y = d3.scaleLinear()
            .domain([2.8, d3.max(filteredData, d => parseFloat(d['AverageGradeReceived']))])
            .range([ height, 0]);
        svg.append("g")
            .call(d3.axisLeft(y));

        // Add a scale for bubble size
        var z = d3.scaleSqrt()
            .domain([0, d3.max(filteredData, d => parseFloat(d['StudyHoursperWeek']))])
            .range([1, 30]);

        
        // -1- Create a tooltip div that is hidden by default:
        var tooltip = d3.select("#bubble_plot")
            .append("div")
            .style("opacity", 0)
            .attr("class", "tooltip")
            .style("border", "solid")
            .style("border-width", "2px")
            .style("border-radius", "5px")
            .style("padding", "10px") 
            .style("text-align", "center")

            
        // -2- Create 3 functions to show / update (when mouse move but stay on same circle) / hide the tooltip
        var showTooltip = function(d) {

            tooltip
                .transition()
                .duration(200)
            tooltip
            .style("opacity", 1)
            d3.selectAll(".dot")
                .transition()
                .duration(30)
                .style("fill", "lightgrey")
                .attr("r", 100)
        }
        var moveTooltip = function(event, d) {
            tooltip.html(`
                <div style="font-size: 20px; font-weight: bold; margin-bottom: 5px;">${d.CourseID}</div>
                <div style="font-size: 16px;">
                    <p style="margin: 2px 0;"><span style="font-weight: bold;">The average study hour per week is:</span> ${d.StudyHoursperWeek}</p>
                    <p style="margin: 2px 0;"><span style="font-weight: bold;">The average grade received is: :</span> ${d.AverageGradeReceived}</p>
                </div>
            `)
            .style("left", offsetX + "px")
            .style("top", offsetY + "px")
        }
        var hideTooltip = function(d) {
            tooltip
            .transition()
            .duration(200)
            .style("opacity", 0)
        }
        

        //Options for bubbles
        let rangecolors = [
        "#FB8AAC","#EF8CB7","#E08FC0","#CF93C7","#BC96CC",
        "#A89ACF","#939DCF","#7D9FCC","#67A1C8","#52A2C0",
        "#3EA3B7","#2DA3AC","#23A29F","#23A092","#2C9E84",
        "#399C76","#459868","#52955B","#5D914F","#688D44",
        "#73883B"];//,"#7D8334"];
        //"#867D2F",
        //"#8E782D","#95722E","#9B6C30","#A06635","#A3603A"];
        
        
        // Add a scale for bubble color
        var myColor = d3.scaleOrdinal()
            .domain(Array.from(new Set(filteredData.map(d => d.CourseID))))
            .range(rangecolors);
        //console.log(Array.from(new Set(filteredData.map(d => d.CourseID))));

        svg.append("text")
            .attr("text-anchor", "end")
            .attr("x", width/2 + margin.left)
            .attr("y", height + margin.top - 20)
            .text('Class Size');

        // Y-axis label
        svg.append("text")
            .attr("text-anchor", "end")
            .attr("transform", "rotate(-90)")
            .attr("y", -margin.left + 20)
            .attr("x", -margin.top -  height/2 + 120)
            .text('Average Grades Received')

        // Add dots
        svg.append('g')
            .selectAll("dot")
            .data(filteredData)
            .enter()
            .append("circle")
            .attr("class", "bubbles")
            .attr("cx", function (d) { return x(parseFloat(d['TotalEnrolledinCourse'])); } )
            .attr("cy", function (d) { return y(parseFloat(d['AverageGradeReceived'])); } )
            .attr("r", function (d) { 
                return z(parseFloat(d.StudyHoursperWeek)); } )
.style("fill", function(d) { 
                let courseNumber = parseInt(d.CourseID.replace("DSC", "").trim());
                if (courseNumber >= 100) {
                    return "blue"; // Color for upper division courses
                } else {
                    return "red"; // Color for lower division courses
                }
            })
            .attr("opacity", 0.7)
            .on("mouseover", showTooltip )
            .on("mousemove", moveTooltip )
            .on("mouseleave", hideTooltip )
        }
        


    onMount(() => {
        fetch('./capes_dsc_clean_v1.csv')
            .then(response => response.text())
            .then(csv => {
                let data = d3.csvParse(csv);

                let allCourses = new Set(data.map(d => d.CourseID));
                allCourses.add("All Courses")

                createBubble(data, "All Courses");

                })

            
            .catch(error => {
                console.error('Error loading data:', error);
            });
        });

</script>

<div id="bubble_plot" style ="position: relative">

</div>