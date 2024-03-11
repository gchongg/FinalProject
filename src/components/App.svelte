<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import Scrolly from "./Scrolly.svelte";
    import Scatterplot from "./scatterplot.svelte";
    import Bubbleplot from './bubbleplot.svelte';

    let data = [];
    let value;
    const steps = ["Study Time Analysis", "Average Grades with Respect to ", "Challenges & Insights"];

    onMount(async () => {
        data = await d3.csv('/capes_dsc_clean_v1.csv', d3.autoType);
    });
</script>

<div class="info">
    <h2>Exploring Academic Success in Data Science Courses: A Visual Analysis of Study Patterns and Grades from CAPEs at UC San Diego</h2>
    <p>
        Dive into an immersive visual journey exploring the intricate relationship between study time and academic performance in Data Science courses at UC San Diego. Discover how students' study habits correlate with their grades and unveil the hidden challenges of interpreting educational data.
	</p>
</div>

<section>
    <div class="spacer"></div>

    <Scrolly bind:value>
        {#each steps as step, i}
        <div class="step" class:active={value === i}>
            <h3>{step}</h3>
            <div class="text-content">
                {#if (step === 'Study Time Analysis')}
                    <p>An in-depth look at how study time impacts grades.</p>
                    <Scatterplot></Scatterplot>
                {/if}
                {#if (step === 'Average Grades vs. Class Size')}
                    <p>Visualizing the distribution of grades across various study times.</p>
                    <Bubbleplot></Bubbleplot>
                {/if}
                {#if (step === 'Challenges & Insights')}
                    <p>Exploring the complexities of data interpretation in educational analytics.</p>
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

    /* mark {
        background-color: #84fab0;
        color: #333;
        padding: 0.5em;
        border-radius: 4px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.2);
        display: inline-block;
        margin-top: 1em;
    } */

    /* Section and Steps */
    section {
        padding: 2em 0;
    }

    .spacer {
        height: 15vh;
    }

    .step {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: auto; /* Adjusted for content */
        padding: 40px;
        background: #fff;
        margin: 1em auto;
        text-align: center;
        box-shadow: 0 6px 12px rgba(0,0,0,0.1);
        border-radius: 8px;
        max-width: 800px;
        transition: all 0.3s ease;
        position: relative;
        overflow: hidden; /* Ensure no content spills out */
    }

    .step h3 {
        color: #333;
        position: relative;
        z-index: 2; /* Above the graph */
    }

    .step .text-content {
        position: relative;
        z-index: 2; /* Above the graph */
        padding-bottom: 20px; /* Space between text and graph */
    }

    .step.active {
        background-image: linear-gradient(to right, #fbc2eb 0%, #a6c1ee 100%);
    }

    .step:hover {
        box-shadow: 0 8px 16px rgba(0,0,0,0.25);
    }
</style>
