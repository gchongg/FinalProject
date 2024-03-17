<script>
import * as d3 from 'd3';
import { onMount } from 'svelte';
import Svelecte from 'svelecte';

let events = [];
let showForm = false;
let showForm2 = false;
let newEvent = {
    days: [],
    start: '',
    end: '',
    title: ''
};

let departments = [];
let courses = [];
let selectedDept = '';
$: formattedDepartments = departments.map(dept => ({
    value: dept,
    text: dept
}));
$: formattedCourses = selectedDept ?
    uniqueCoursesForDept(selectedDept).map(({
        courseName
    }) => ({
        value: courseName,
        text: courseName
    })) :
    [];
$: selectedDept, formattedCourses = selectedDept ?
    uniqueCoursesForDept(selectedDept).map(({
        courseName
    }) => ({
        value: courseName,
        text: courseName
    })) :
    [];

// Reset selectedCourse whenever selectedDept changes
$: if (selectedDept) selectedCourse = '';

let selectedCourse = '';

let courseList = [];
let totalLectureDuration = 0; // Total time in minutes
let totalStudyHours = 0;
let expectedStudyHours = 0; // Expected study hours for the current course

const daysOfWeek = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
let selectedStartTime = '';
let selectedEndTime = '';

function getRandomColor() {
    // Define a maximum value for RGB components to avoid colors too close to white.
    // Setting this to a value like 200 makes sure the colors are distinguishable from white.
    const maxRGBValue = 200;
    let color = '#';
    for (let i = 0; i < 3; i++) { // There are 3 components: R, G, B
        // Generate a value from 0 to maxRGBValue for each component.
        const component = Math.floor(Math.random() * (maxRGBValue + 1)).toString(16);
        color += component.padStart(2, '0');
    }
    return color;
}

let currentColorIndex = 0;
const colorScheme = d3.schemeCategory10; // This is an array of color strings

function getNextColor() {
    const color = colorScheme[currentColorIndex];
    currentColorIndex = (currentColorIndex + 1) % colorScheme.length; // Loop back to start when reaching the end
    return color;
}

function uniqueCoursesForDept(dept) {
    const filteredCourses = courses.filter(course => course.dept === dept);
    const uniqueCourseNames = new Set(filteredCourses.map(course => course.courseName));
    return Array.from(uniqueCourseNames).map(courseName => ({
        dept,
        courseName
    }));
}

function addEvent() {
    if (!newEvent.title.trim()) {
        alert('Please enter a title for the event.');
        return;
    }

    if (!selectedStartTime || !selectedEndTime) {
        alert('Please specify both start and end times.');
        return;
    }

    if (newEvent.days.length === 0) {
        alert('Please select at least one day.');
        return;
    }

    const color = getRandomColor();
    newEvent.days.forEach(day => {
        events.push({
            ...newEvent,
            day,
            color,
            start: selectedStartTime, // Assuming you'll use selectedStartTime/EndTime for this too
            end: selectedEndTime,
        });
    });

    showForm = false;
    drawEvents();

    newEvent = { days: [], start: '', end: '', title: '' };
    selectedStartTime = '';
    selectedEndTime = '';
}


function clearSchedule() {
    events = []; // Clear the events array
    courseList = [];
    totalLectureDuration = 0; // Total time in minutes
    totalStudyHours = 0;
    expectedStudyHours = '';
    drawEvents() // If you have a function to redraw the calendar, call it here to update the view
}

function addCourseEvent() {
    if (!selectedCourse || selectedCourse === '') {
        alert('Please select a course name.');
        return;
    }

    if (!selectedStartTime || !selectedEndTime) {
        alert('Please specify both start and end times.');
        return;
    }

    if (newEvent.days.length === 0) {
        alert('Please select at least one day for the course.');
        return;
    }

    const courseTitle = courses.find(course => course.courseName === selectedCourse)?.courseName;
    if (!courseTitle) {
        alert('Selected course is invalid.');
        return;
    }

    const eventColor = getNextColor();
    let dailyTimeCommitment = 0;

    newEvent.days.forEach(dayIndex => {
        const startTimeParts = selectedStartTime.split(':').map(Number);
        const endTimeParts = selectedEndTime.split(':').map(Number);
        const timeDifference = (endTimeParts[0] * 60 + endTimeParts[1]) - (startTimeParts[0] * 60 + startTimeParts[1]);

        dailyTimeCommitment += timeDifference;

        events.push({
            title: courseTitle,
            day: dayIndex,
            start: selectedStartTime,
            end: selectedEndTime,
            color: eventColor,
        });
    });

    totalLectureDuration += dailyTimeCommitment;
    const courseData = courses.find(course => course.courseName === selectedCourse);
    courseList = [...courseList, { title: courseTitle, expectedHours: courseData.expectedHours }];
    totalStudyHours += parseFloat(courseData.expectedHours);

    drawEvents(); // Assuming this updates your calendar view

    showForm2 = false;
    newEvent = { days: [], start: '', end: '', title: '' };
    selectedDept = '';
    selectedStartTime = '';
    selectedEndTime = '';
    // To trigger Svelte's reactivity, reassign courseList even if it's logically the same
    courseList = courseList.slice();
}

onMount(async () => {
    drawCalendar();
    const response = await fetch('./capes_grouped.csv');
    const csvText = await response.text();
    const data = d3.csvParse(csvText);

    const deptSet = new Set(data.map(row => row.dept));
    departments = Array.from(deptSet);
    selectedDept = "";

    courses = data.map(row => ({
        dept: row.dept,
        courseName: row.CourseNum,
        expectedHours: row.StudyHoursperWeek
    }));
});

function drawCalendar() {
    const svg = d3.select('#my-calendar').append('svg')
        .attr('width', 800)
        .attr('height', 900)
        .style('background-color', '#f9f9f9')
        .style('box-shadow', '0 0 10px rgba(0,0,0,0.1)');

    const columnWidth = svg.attr('width') / daysOfWeek.length;

    daysOfWeek.forEach((day, i) => {
        svg.append('rect')
            .attr('x', i * columnWidth)
            .attr('y', 0)
            .attr('width', columnWidth)
            .attr('height', svg.attr('height'))
            .attr('fill', 'none')
            .attr('stroke', '#ddd');

        svg.append('text')
            .attr('x', (i * columnWidth) + (columnWidth / 2))
            .attr('y', 20)
            .attr('text-anchor', 'middle')
            .attr('fill', '#333')
            .style('font-family', 'Arial, sans-serif')
            .text(day);
    });
}

function drawEvents() {
    d3.select('#my-calendar svg').selectAll('.event').remove();
    const svg = d3.select('#my-calendar svg');
    const columnWidth = svg.attr('width') / daysOfWeek.length;

    events.forEach(event => {
        const startY = timeToPosition(event.start);
        const endY = timeToPosition(event.end);
        const eventHeight = endY - startY;
        const eventY = startY + 30; // Offset for header space
        const eventX = event.day * columnWidth + 10; // Offset within day column

        svg.append('rect')
            .attr('class', 'event')
            .attr('x', eventX)
            .attr('y', eventY)
            .attr('width', columnWidth - 20)
            .attr('height', eventHeight)
            .attr('fill', event.color)
            .attr('opacity', 0.75)
            .attr('rx', 10)
            .attr('ry', 10);

        svg.append('text')
            .attr('class', 'event')
            .attr('x', eventX + (columnWidth - 20) / 2)
            .attr('y', eventY + eventHeight / 2)
            .attr('text-anchor', 'middle')
            .attr('fill', 'white')
            .attr('dominant-baseline', 'middle')
            .text(event.title);
    });
}

function timeToPosition(time) {
    const [hours, minutes] = time.split(':').map(Number);
    return (hours * 60 + minutes) * (800 / 1440);
}
</script>

<div class="main-container">
    <div class="left-view">
        <div class="dashboard-content">
            <h1> Expected Weekly Time Commitments</h1>
            <div class="dashboard-section">
                <p align="left" >Expected Lecture Hours: &nbsp;{Math.floor(totalLectureDuration / 60)} hours and {totalLectureDuration % 60} minutes</p>
                <p align="left">Expected Study Hours: &nbsp;&nbsp;&nbsp; {totalStudyHours} hours</p>
            </div>
            <div class="dashboard-section courses-list">
                <h2>Enrolled Courses</h2>
                {#if courseList.length > 0}
                <ul>
                    {#each courseList as { title, expectedHours }}
                    <li>
                        <span class="course-title">{title}</span>
                        <span class="expected-hours">Expected Study Hours: {expectedHours}</span>
                    </li>
                    {/each}
                </ul>
                {/if}
            </div>
        </div>
        <div class="controls-container">
            <button on:click="{() => { showForm = !showForm; showForm2 = false; }}">Add Event</button>
            <button on:click="{() => { showForm2 = !showForm2; showForm = false; }}">Add Course</button>
            <button on:click={clearSchedule}>Clear Schedule</button>
        </div>
        {#if showForm2}
        <div class="event-form">
            <div>
                <label for="deptSelect">Department</label>
                <Svelecte bind:value="{selectedDept}" options="{formattedDepartments}" placeholder="Select a Department" />
            </div>

            <div>
                <label for="courseSvelecte">Course Name</label>
                <Svelecte bind:value="{selectedCourse}" options="{formattedCourses}" placeholder="Select a course" />
            </div>

            <div class="days-of-week">
                {#each daysOfWeek as day, index}
                <div class="checkbox-group">
                    <input type="checkbox" id="day{index}" bind:group="{newEvent.days}" value="{index}">
                    <label for="day{index}">{day}</label>
                </div>
                {/each}
            </div>

            <div class="time-inputs">
                <label for="startTimeInput">Start Time:</label>
                <input id="startTimeInput" type="time" bind:value="{selectedStartTime}" placeholder="HH:MM">

                <label for="endTimeInput">End Time:</label>
                <input id="endTimeInput" type="time" bind:value="{selectedEndTime}" placeholder="HH:MM">
            </div>

            <button on:click="{addCourseEvent}">Submit</button>
        </div>
        {/if}
        {#if showForm}
        <div class="event-form">
            <div>
                <label for="Title">Title</label>
                <input id="eventTitle" type="text" bind:value="{newEvent.title}" placeholder="Input Event Title" />
            </div>

            <div class="days-of-week">
                {#each daysOfWeek as day, index}
                <div class="checkbox-group">
                    <input type="checkbox" id="day{index}" bind:group="{newEvent.days}" value="{index}">
                    <label for="day{index}">{day}</label>
                </div>
                {/each}
            </div>

            <div class="time-inputs">
                <label for="startTimeInput">Start Time:</label>
                <input id="startTimeInput" type="time" bind:value="{selectedStartTime}" placeholder="HH:MM">

                <label for="endTimeInput">End Time:</label>
                <input id="endTimeInput" type="time" bind:value="{selectedEndTime}" placeholder="HH:MM">
            </div>

            <button on:click="{addEvent}">Submit</button>
        </div>
        {/if}

    </div>

    <div class="calendar-container" id="my-calendar">
    </div>
</div>

<style>
.main-container {
    display: grid;
    grid-template-columns: auto minmax(0, 1fr);
    grid-template-rows: auto 1fr;
    grid-template-areas:
        "controls calendar"
        "dashboard calendar";
    gap: 20px;
    padding: 20px;
    max-width: 100%;
    margin: auto;
}

.controls-container {
    grid-area: controls;
    padding: 20px
        /* Style as needed */
}

.dashboard-content {
    grid-area: dashboard;
    background-color: #fff;
    padding: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    min-height: 200px;
    /* Adjust as needed */
}

.dashboard-section {
    margin-bottom: 20px;
}

.dashboard-section:last-child {
    margin-bottom: 0;
}

.dashboard-section h2 {
    margin-bottom: 10px;
    color: #333;
    font-size: 20px;
}

.dashboard-section p,
.dashboard-section ul {
    color: #666;
    font-size: 16px;
    margin: 5px 0;
}

.courses-list ul {
    list-style: none;
    padding: 0;
}

.courses-list li {
    background-color: #f9f9f9;
    border: 1px solid #e1e1e1;
    border-radius: 4px;
    padding: 10px;
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
}

.course-title {
    font-weight: bold;
    color: #333;
}

.expected-hours {
    font-style: italic;
}

.calendar-container {
    grid-area: calendar;
    background-color: #fff;
    padding: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
}

button {
    background-color: #007bff;
    color: white;
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.3s;
    margin-right: 10px;
    /* Spacing between buttons */
}

button:hover {
    background-color: #0056b3;
}

button:focus {
    outline: none;
    box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.5);
}

.event-form {
    background-color: #f7f7f7;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    display: grid;
    gap: 16px;
    max-width: 600px;
}

.event-form input[type="text"],
.event-form select {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    width: 100%;
}

.event-form button {
    background-color: #007bff;
    color: white;
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.event-form button:hover {
    background-color: #0056b3;
}
</style>
