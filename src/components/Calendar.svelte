<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

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
  let selectedCourse = '';

  let courseList = [];
  let totalLectureDuration = 0; // Total time in minutes
  let totalStudyHours = 0;
  let expectedStudyHours = ''; // Expected study hours for the current course

  const daysOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
  const timeOptions = generateTimeOptions();

  function generateTimeOptions() {
    const options = [];
    for (let i = 0; i <= 24; i++) {
      options.push(`${i.toString().padStart(2, '0')}:00`);
      if (i < 24) options.push(`${i.toString().padStart(2, '0')}:30`);
    }
    return options;
  }

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
    return Array.from(uniqueCourseNames).map(courseName => ({ dept, courseName }));
  }

  function addEvent() {
    const color = getRandomColor();
    newEvent.days.forEach(day => {
      events.push({ ...newEvent, day, color });
    });
    showForm = false;
    drawEvents();

    newEvent = { days: [], start: '', end: '', title: '' };
  }

  function addCourseEvent() {

    const courseTitle = courses.find(course => course.courseName === selectedCourse)?.courseName || 'Unnamed Course';
    const eventColor = getNextColor(); // Generate a color for this course
    // Assuming courseData is already defined as the data for the selected course...
    const courseData = courses.find(course => course.courseName === selectedCourse);

    // Check if the course is already in the list to avoid duplicates
    const existingCourse = courseList.find(course => course.title === courseData.courseName);
    if (!existingCourse) {
        courseList = [
            ...courseList,
            { title: courseData.courseName, expectedHours: courseData.expectedHours }
        ];
    }

    let dailyTimeCommitment = 0;

    expectedStudyHours = courseData && !isNaN(courseData.expectedHours) ? parseFloat(courseData.expectedHours) : 0;

    totalStudyHours += expectedStudyHours;

    newEvent.days.forEach(dayIndex => {
        const startTimeParts = newEvent.start.split(':').map(Number);
        const endTimeParts = newEvent.end.split(':').map(Number);
        const startTimeMinutes = startTimeParts[0] * 60 + startTimeParts[1];
        const endTimeMinutes = endTimeParts[0] * 60 + endTimeParts[1];
        const timeDifference = endTimeMinutes - startTimeMinutes;

        dailyTimeCommitment += timeDifference; // Add time difference for this day to the daily time commitment

        const courseEvent = {
            title: courseTitle,
            day: dayIndex,
            start: newEvent.start,
            end: newEvent.end,
            color: eventColor,
        };
        events.push(courseEvent);
    });

    totalLectureDuration += dailyTimeCommitment; // Update the total time commitment with this course's commitment

    showForm2 = false; // Hide the course form after adding the event(s)
    drawEvents(); // Redraw the calendar with the new events

    // Update dashboard with expected study hours for this course


    // Update dashboard with total time commitment
    updatetotalLectureDurationDisplay();

    // Reset form fields for courses
    newEvent = { days: [], start: '', end: '', title: '' };
    selectedDept = departments.length > 0 ? departments[0] : '';
    selectedCourse = ''; // Reset selected course
}

  function updatetotalLectureDurationDisplay() {
      const hours = Math.floor(totalLectureDuration / 60);
      const minutes = totalLectureDuration % 60;
      document.getElementById('totalLectureDurationDisplay').textContent = `Total Time Commitment: ${hours} hours and ${minutes} minutes`;
  }



  onMount(async () => {
    drawCalendar();
    const response = await fetch('./capes.csv');
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

<style>
  button {
    background-color: #007bff;
    color: white;
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.3s;
  }

  button:hover {
    background-color: #0056b3;
  }

  button:focus {
    outline: none;
    box-shadow: 0 0 0 2px rgba(76,175,80,0.5);
  }

  .event-form {
    background-color: #f7f7f7;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    display: grid;
    gap: 16px;
    max-width: 600px;
  }

  .event-form input[type="text"], .event-form select {
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

  .days-of-week, .time-selectors {
    display: flex;
    gap: 10px;
  }

  .checkbox-group {
    display: flex;
    align-items: center;
  }

  label {
    margin-right: 8px;
    font-size: 14px;
  }

  .dashboard-container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
    margin: 20px;
  }

  #event-visualizations {
    background-color: #f0f0f0;
    padding: 20px;
    box-shadow: 0 0 10px rgba(0,0,0,0.1);
  }
  .dashboard {
    background-color: #f9f9f9;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
    max-width: 600px;
    margin: auto;
  }

  .dashboard h2 {
    color: #333;
    font-size: 20px;
    margin-bottom: 16px;
  }

  .dashboard ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .dashboard ul li {
    background-color: #ffffff;
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 10px 15px;
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: all 0.3s ease;
  }

  .dashboard ul li:hover {
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transform: translateY(-2px);
  }

  .course-title {
    font-weight: bold;
  }

  .expected-hours {
    color: #666;
    font-style: italic;
  }
</style>

<button on:click="{() => { showForm = !showForm; showForm2 = false; }}">Add Event</button>
<button on:click="{() => { showForm2 = !showForm2; showForm = false; }}">Add Course</button>


{#if showForm2}
  <div class="event-form">
    <div>
      <label for="deptSelect">Department</label>
      <select id="deptSelect" bind:value="{selectedDept}">
        {#each departments as dept}
          <option value="{dept}">{dept}</option>
        {/each}
      </select>
    </div>
    
    <div>
      <label for="courseSelect">Course Name (Optional)</label>
      <select id="courseSelect" bind:value="{selectedCourse}">
        {#if selectedDept}
          {#each uniqueCoursesForDept(selectedDept) as { courseName }}
            <option value="{courseName}">{courseName}</option>
          {/each}
        {/if}
      </select>
    </div>
    
    <div class="days-of-week">
      {#each daysOfWeek as day, index}
        <div class="checkbox-group">
          <input type="checkbox" id="day{index}" bind:group="{newEvent.days}" value="{index}">
          <label for="day{index}">{day}</label>
        </div>
      {/each}
    </div>
    
    <div class="time-selectors">
      <div>
        <label for="startTime">Start Time</label>
        <select id="startTime" bind:value="{newEvent.start}">
          {#each timeOptions as option}
            <option value="{option}">{option}</option>
          {/each}
        </select>
      </div>
      
      <div>
        <label for="endTime">End Time</label>
        <select id="endTime" bind:value="{newEvent.end}">
          {#each timeOptions as option}
            <option value="{option}">{option}</option>
          {/each}
        </select>
      </div>
    </div>
    
    <button on:click="{addCourseEvent}">Submit</button>
  </div>
{/if}

{#if showForm}
  <div class="event-form">
    <div>
      <label for="Title">Title</label>
      <input id="eventTitle" type="text" bind:value="{newEvent.title}" placeholder="Event Title" />
    </div>
    
    <div class="days-of-week">
      {#each daysOfWeek as day, index}
        <div class="checkbox-group">
          <input type="checkbox" id="day{index}" bind:group="{newEvent.days}" value="{index}">
          <label for="day{index}">{day}</label>
        </div>
      {/each}
    </div>
    
    <div class="time-selectors">
      <div>
        <label for="startTime">Start Time</label>
        <select id="startTime" bind:value="{newEvent.start}">
          {#each timeOptions as option}
            <option value="{option}">{option}</option>
          {/each}
        </select>
      </div>
      
      <div>
        <label for="endTime">End Time</label>
        <select id="endTime" bind:value="{newEvent.end}">
          {#each timeOptions as option}
            <option value="{option}">{option}</option>
          {/each}
        </select>
      </div>
    </div>
    
    <button on:click="{addEvent}">Submit</button>
  </div>
{/if}

<div class="dashboard-container">

  <div id="my-calendar"></div>

  <div id="event-visualizations">
    <h1>Dashboard</h1>
    <!-- Visualization content goes here -->
    <!-- Display for expected study hours -->
    <div>Expected Study Hours: {expectedStudyHours}</div>
    <!-- Display for total time commitment -->

    <div>Total Time Commitment: {Math.floor(totalLectureDuration / 60)} hours and {totalLectureDuration % 60} minutes</div>

    <div>Expected Study Hours: {totalStudyHours}</div>

    <div class="dashboard">
      <h2> Enrolled Courses</h2>
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
 
</div>
