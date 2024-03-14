<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
  
    let events = [];
    let showForm = false;
    let newEvent = {
        days: [], // Now an array to support multiple days
        start: '',
        end: '',
        title: ''
    };
  
    const daysOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
    const timeOptions = generateTimeOptions();
  
    function generateTimeOptions() {
      const options = [];
      for (let i = 0; i < 24; i++) {
        options.push(`${i.toString().padStart(2, '0')}:00`);
        options.push(`${i.toString().padStart(2, '0')}:30`);
      }
      return options;
    }
    function getRandomColor() {
        const letters = '0123456789ABCDEF';
        let color = '#';
        for (let i = 0; i < 6; i++) {
            color += letters[Math.floor(Math.random() * 16)];
        }
        return color;
    }

    function addEvent() {
        const color = getRandomColor(); // Generate a color for the new event once

        newEvent.days.forEach(day => {
            events.push({
                day: day,
                start: newEvent.start,
                end: newEvent.end,
                title: newEvent.title,
                color: color // Use the same color for all days
            });
        });
        showForm = false;
        drawEvents(); // Redraw events with the new addition
    }


  
    onMount(() => {
      drawCalendar();
      drawEvents();
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
    // Remove existing event elements to prevent duplicates
    d3.select('#my-calendar svg').selectAll('.event').remove();

    const svg = d3.select('#my-calendar svg');
    const columnWidth = svg.attr('width') / daysOfWeek.length;

    events.forEach(event => {
        // Convert start and end times to pixel positions
        const startY = timeToPosition(event.start);
        const endY = timeToPosition(event.end);
        const eventHeight = endY - startY;

        // Adjust for visual layout
        const eventY = startY + 30; // Offset for header space
        const eventX = event.day * columnWidth + 10; // Offset within day column

        // Append the event rectangle
        svg.append('rect')
            .attr('class', 'event')
            .attr('x', eventX)
            .attr('y', eventY)
            .attr('width', columnWidth - 20) // Padding within the column
            .attr('height', eventHeight)
            .attr('fill', event.color)
            .attr('opacity', 0.75)
            .attr('rx', 10)
            .attr('ry', 10);  



        // Append the event title
        svg.append('text')
            .attr('class', 'event')
            .attr('x', eventX + (columnWidth - 20) / 2) // Center text in the event rect
            .attr('y', eventY + eventHeight / 2) // Vertically center text
            .attr('text-anchor', 'middle')
            .attr('fill', 'white')
            .attr('dominant-baseline', 'middle') // Ensure text is centered
            .text(event.title);
        });
    }
    function timeToPosition(time) {
      const [hours, minutes] = time.split(':').map(Number);
      return (hours * 60 + minutes) * (800/1440); // Assuming 60px height per hour, adjust if necessary
    }
  </script>
  <style>
    button {
      background-color: #007bff; /* Green background */
      color: white; /* White text */
      padding: 12px 24px; /* Top and bottom padding, left and right padding */
      border: none; /* No border */
      border-radius: 4px; /* Rounded corners */
      cursor: pointer; /* Hand cursor on hover */
      font-size: 16px; /* Larger font size */
      transition: background-color 0.3s; /* Smooth transition for hover effect */
    }

    button:hover {
      background-color: #0056b3; /* Darker green background on hover */
    }

    /* Optional: focus state for accessibility */
    button:focus {
      outline: none; /* Removes the outline */
      box-shadow: 0 0 0 2px rgba(76,175,80,0.5); /* Adds a green glow */
    }

    .event-form {
    background-color: #f7f7f7;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    display: grid;
    gap: 16px;
    max-width: 400px;
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

    .days-of-week {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
    }

    .time-selectors {
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
    .grid-container {
      display: grid; /* Establishes a grid container */
      grid-template-columns: 3fr 1fr; /* Allocates 3 parts of the space for the calendar and 1 part for the dashboard */
      column-gap: 20px; /* Optional: Specifies a gap between the columns */
      height: 100vh; /* Adjust the height as needed */
      /* Additional styling as needed */
    }

    #my-calendar {
      /* Calendar-specific styles */
      /* No need to set width, as it's controlled by grid-template-columns */
      background-color: #eef; /* Example styling */
      /* Adjust padding, margin, or other styles as needed */
    }

    .dashboard {
      /* Dashboard-specific styles */
      background-color: #ffe; /* Example styling */
      overflow-y: auto; /* Makes the dashboard scrollable if content exceeds its height */
      /* Adjust padding, margin, or other styles as needed */
    }
  </style>
  <button on:click="{() => showForm = !showForm}">Add Event</button>
    
  {#if showForm}
  <div class="event-form">
    <div>
      <label for="eventTitle">Event Title</label>
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

  <div class="grid-container">
    <div id="my-calendar">
      <!-- Calendar content goes here -->
    </div>

    <div class="dashboard">
      <h2>Dashboard</h2>
      <!-- Dashboard content -->
      <p>Event stats, upcoming events, etc.</p>
    </div>
  </div>
    

  