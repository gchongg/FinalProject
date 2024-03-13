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
  
  <button on:click="{() => showForm = !showForm}">Add Event</button>
  
{#if showForm}
    <div>
        <input type="text" bind:value="{newEvent.title}" placeholder="Event Title" />
        {#each daysOfWeek as day, index}
            <input type="checkbox" bind:group="{newEvent.days}" value="{index}"> {day}
        {/each}
        <select bind:value="{newEvent.start}">
        {#each timeOptions as option}
          <option value="{option}">{option}</option>
        {/each}
        </select>
        <select bind:value="{newEvent.end}">
        {#each timeOptions as option}
          <option value="{option}">{option}</option>
        {/each}
        </select>
        <button on:click="{addEvent}">Submit</button>
    </div>
  {/if}
  
  <div id="my-calendar"></div>
  