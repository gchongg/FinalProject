<script>
import * as d3 from 'd3';
import {
    onMount
} from 'svelte';
import Scrolly from "./Scrolly.svelte";
import Scatterplot from "./scatterplot.svelte";
import Bubbleplot from './bubbleplot.svelte';
import Calendar from './Calendar.svelte';
import Histogram from './Histogram.svelte';

let data = [];
let value;
const steps = ["Study Hour and Academic Performance", 'Average Grades vs. Class Size', "View Course Distribution" ,"Create Your Own Course Schedule", "Takeaway"];

onMount(async () => {
    data = await d3.csv('/capes_dsc_clean_v1.csv', d3.autoType);
});
</script>

<div class="info">
    <h2>Unveiling Academic Success in Data Science: An Interactive Analysis of Study Patterns and Grades from UC San Diego's CAPEs</h2>
    <p>
        Concerned about managing your time and aiming for a successful academic quarter? Join us on an interactive expedition into the heart of academic achievement! We open the book on Data Science courses at UC San Diego, shedding light on the intricate dance between study time and grades.
        Our exploratory website uses data from CAPEs, which can be found here. We filtered the dataset to include only Data Science courses, with reports ranging from 2017 to 2023.
    </p>
    <p>
        Our exploratory website uses data from CAPEs, which can be found here. We filtered the dataset to include only Data Science courses, with reports ranging from 2017 to 2023.
    </p>
</div>

<section>
    <Scrolly bind:value>
        {#each steps as step, i}
        <div class="step" class:active={value === i}>
            <h3>{step}</h3>
            <div class="text-content">
                {#if (step === 'Study Hour and Academic Performance')}
                <p>in-depth look at how study time impacts grades.</p>
                <body>
                    <p>Let's start with debunking the myth that upper-division courses are harder than lower-division ones. The following visualization shows that students generally dedicate more study hours to lower-division courses.</p>
                    
                    <p>Here's a fun fact: Do you remember taking DSC 30 and feeling overwhelmed? It turns out that this course is among the most challenging in the Data Science curriculum. It demands more study time, yet it has the lowest average grades.</p>
                </body>
                <Scatterplot></Scatterplot>
                <p class = "instruction"> Hover over points to see the course information of corresponding point.</p>

                <p> From the Capes dataset, students tend to spend more time studying for lower division data science courses
                    than those of upper division. Furthermore, lower division data science course tends
                    to be more difficult for most students. DSC30 and DSC80 tends to demand the most time from
                    students yet having the lowest average grades.
                </p>
                {/if}
                {#if (step === 'Average Grades vs. Class Size')}
                <p>Visualizing the distribution of grades across various study times.</p>
                <body>
                    <p>
                        So, what's the reason? Does the class size make upper-division courses more suited for the quarter system, thereby enhancing student learning?
                    </p>
                    <p>
                        Interestingly, it appears that larger class sizes often correlate with lower average grades. However, not all upper-division courses have smaller class sizes, yet they still maintain a higher average grade compared to lower-division courses.
                    </p>
                </body>
                <p class = "instruction"> Hover over points to see the course information of corresponding point.</p>
                <Bubbleplot></Bubbleplot>
                <p> From the Capes dataset, DSC courses with bigger class sizes often have a lower
                    average grade. From this visualization, DSC20 and DSC30 seem to be weeder class within
                    the data science catalog.
                </p>
                <p>
                    While it's challenging to generalize the number of hours a Data Science student should dedicate per course. We've addressed this issue by building a tool for time management for all students. It allows you to visualize the time you will likely need to allocate to your UCSD courses to pass successfully, based on the average study hours reported by students in the past.
                </p>
                <p> 
                    We invite you to try out our scheduling feature! Add your planned course lectures, discussions, as well as personal events like lunch breaks. Get a feel for your potential quarter and see if it sets you up for a thriving academic term or suggests a risk of burnout. Give it a try and optimize your academic journey! 
                </p>
                {/if}
            

                {#if (step === 'View Course Distribution')}
                    <Histogram></Histogram>
                {/if}


                {#if (step === 'Create Your Own Course Schedule')}
                <Calendar></Calendar>
                {/if}


                {#if (step === 'Takeaway')}
                    <body>
                        <p>The one thing everyone should learn from our project is that success in academia is not merely about hard work, but about working smart by efficiently allocating study hours. Our project succeeds in explaining this by providing an interactive, user-friendly tool that uses real data to assist students in managing their time effectively.</p>
                    </body>
                {/if}
            </div>
        </div>
        {/each}
    </Scrolly>
    <div class="spacer"></div>
</section>

<style>
/* Global Styles */
/* .body {
        background-color: #fafafa;
        color: #333;
        font-family: 'Roboto', sans-serif;
        margin: 0;
        padding: 0;
    } */

/* Information Section */
.info {
    font-size: 20px;
    color: #444;
    max-width: 900px;
    margin: 2em auto;
    padding: 1.5em;
    background-color: #ffffff;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    background-image: linear-gradient(135deg, #f6d365 0%, #fda085 100%);
    text-align: center;
}

h2 {
    margin-top: 0;
    color: #333;
}

section {
    padding: 2em 0;
}

.spacer {
    height: 15vh;
}

.step {
    font-size: 18px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: auto;
    /* Adjusted to automatically accommodate content */
    padding: 40px;
    background: #fff;
    margin: 1em auto;
    text-align: center;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    max-width: 1400px;
    transition: all 0.3s ease;
    position: relative;
    overflow: visible;
    /* Consider changing if content is still cut off */
}

.instruction {
    color: purple;
}

.step h3 {
    color: #333;
    position: relative;
    z-index: 2;
    /* Above the graph */
}

.step .text-content {
    position: relative;
    z-index: 2;
    /* Above the graph */
    padding-bottom: 20px;
    /* Space between text and graph */
}

.step.active {
    background-image: linear-gradient(to right, #fbc2eb 0%, #a6c1ee 100%);
}

.step:hover {
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.25);
}

/* Global Styles */
body {
    color: #333;
    font-family: 'Roboto', sans-serif;
    margin: 0;
    padding: 0 2%;
    /* Increased left and right padding for the entire body */
    overflow-x: hidden;
    /* Prevents horizontal scroll */
}

/* Information Section */
.info {
    font-size: 22px;
    /* Increased font size */
    color: #444;
    max-width: 90%;
    /* Adjusted for increased margin */
    margin: 2em auto;
    padding: 1.5em;
    background-color: #ffffff;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    background-image: linear-gradient(135deg, #f6d365 0%, #fda085 100%);
    text-align: center;
}

h2,
h3 {
    margin: 0.5em 0;
    color: #333;
    font-weight: bold;
    /* Makes headers bold */
    line-height: 1.2;
    /* Adjusts line height for better readability */
    font-size: 1.5em;
    /* Increased font size for headers */
}

p {
    font-size: 18px;

    /* Increased font size for paragraphs */
    line-height: 1.6;
    /* Improves readability */
    margin-bottom: 1em;
    /* Adds space below paragraphs */
}

/* Responsive Design */
@media (max-width: 768px) {

    .info,
    .step {
        padding: 1em;
        font-size: 20px;
        /* Adjusts font size for smaller screens */
    }

    h2,
    h3 {
        font-size: 1.25em;
        /* Adjusts header font size for smaller screens */
    }

    p {
        font-size: 16px;
        /* Adjusts paragraph font size for smaller screens */
    }
}
</style>
