<svelte:head>
	<title>Tutorial</title>
	<meta name="description" content="About this app" />
</svelte:head>

<script>
	import Scrolly from "../helpers/Scrolly.svelte";
	let value = $state();
</script>

<section id="scrolly">
	<h2>Scrolly <span>{value || "-"}</span></h2>
	<div class="spacer"></div>
	<Scrolly bind:value>
		{#each [0, 1, 2, 3, 4] as text, i}
			{@const active = value === i}
			<div class="step" class:active>
				<p>{text}</p>
			</div>
		{/each}
	</Scrolly>
	<div class="spacer"></div>
</section>

<style>
	h2 {
		position: sticky;
		top: 4em;
	}

	.spacer {
		height: 75vh;
	}

	.step {
		height: 80vh;
		background: var(--color-gray-100);
		text-align: center;
	}

	.step p {
		padding: 1rem;
	}
</style>


<!-- <script>
	import { run } from 'svelte/legacy';

	// @ts-nocheck
	import * as d3 from "d3";

    import { test_elem_1, test_elem_2 } from '../data/babynames';  
	import { combElems, RTD, myDiamond, wordShift_dat } from "allotaxonometer"
	
	import Hero from "../components/Hero.svelte";
	import { range, max } from 'd3'
	
    import Diamond from '../components/DiamondChart.svelte';
    import Wordshift from '../components/WordshiftChart.svelte';
    import DivergingBarChart from '../components/DivergingBarChart.svelte'
    import BarChartRank from '../components/barChart_rank.svelte'
  
	import Scrolly from "../helpers/Scrolly.svelte"
	
	
	const margin = { inner: 160, diamond: 40 };
    
    let DashboardHeight = 815;
    let DashboardWidth = 1200;
    
    // Diamond plot width and height 
    let DiamondHeight = 600;
	let DiamondWidth = DiamondHeight;

    let DiamondInnerHeight = $derived(DiamondHeight - margin.inner);   
    
	// We give some space for the marks in the diamond plot
    // so that cells are not too close to the border
    let trueDiamondHeight = $derived(DiamondInnerHeight - margin.diamond);

    // Wrangling data
    let alpha = $state(0.58);
	
	let me = $derived(combElems(test_elem_1, test_elem_2));
    let rtd = $derived(RTD(me, alpha));
    let dat = $derived(myDiamond(me, rtd));
    let diamond_count = $derived(dat.counts);
	let diamond_dat = $derived(diamond_count.filter(d => d.types !== ""))
    
	let barData = $derived(wordShift_dat(me, dat).slice(0, 30));

	let maxlog10 = $derived(Math.ceil(d3.max([Math.log10(d3.max(me[0].ranks)), Math.log10(d3.max(me[1].ranks))])))
	
	let currentStep = $state(0);
  
	let initialData = $derived(diamond_dat);
	let renderedData = $state(initialData);
	
	let initialBarData = $derived(barData);
	let rendererBarData = $state(initialBarData);
	
	// Series of if-else to play with data.
	run(() => {
	  if (currentStep === 0) {
		renderedData = initialData.map((d => 
		  ({...d, 
			  x1: Math.ceil(d.coord_on_diag),
			  y1: Math.ceil(d.coord_on_diag),
			  value: null,
			  types: ""
			})));
		
		rendererBarData = initialBarData.map((d => 
		  ({...d, 
			  metric: 0,
			})));
  
	  } else if (currentStep === 1) {
		renderedData = initialData.map((d => 
		  ({...d, 
			  x1: Math.ceil(d.coord_on_diag),
			  y1: Math.ceil(d.coord_on_diag),
			  types: ""
			})));
		
		rendererBarData = initialBarData.map((d => 
		  ({...d, 
			  metric: 0
			})));
  
	  } else if (currentStep === 2) {
		renderedData = initialData.map((d => 
			({...d, 
			  x1: d.which_sys == "left" ? Math.ceil(d.coord_on_diag) : d.x1, 
			  y1: d.which_sys == "left" ? Math.ceil(d.coord_on_diag) : d.y1,
			  types: d.which_sys == "left" ? "" : d.types,
			})));
  
		rendererBarData = initialBarData.map((d => 
			  ({...d, 
				  metric: d.metric < 0 ? 0 : d.metric,
				})));
  
	  } else if (currentStep === 3) {
		renderedData = initialData;
		rendererBarData = initialBarData;
	  }
	});
  
</script>
   
<main>
	<h1>Allotaxonometry for all</h1>
	<h2>Boys 1895 vs 1930</h2>
	<p>Here is the rank of the babynames for 1895</p>
	<div class="initial-chart">
	  <BarChartRank barData={test_elem_1.slice(0,30)} height={300}/> 
	</div>
	<p>Here is the ranking for boy babynames in the US for 1930</p>
	<BarChartRank barData={test_elem_2.slice(0,30)} height={300}/> 
	<p>If you wanted to compare which babyname got more popular over time, how would you do it? 
		You can say that Robert is more popular than John. Then what?</p>

	<section>
		<div class='sticky'>
		<div class="dashboard">
			<svg id="mysvg" height={DashboardHeight} width={DashboardWidth}>
				<g class='inner-chart' transform="translate(220, 0)">
					<Diamond {diamond_count} {diamond_dat} {DiamondInnerHeight} {margin} {trueDiamondHeight} {alpha} {maxlog10} {rtd}/>
				</g>
			</svg>
		</div>
	</div>
	<div class="steps">
		<Scrolly bind:value={currentStep}>
		  {#each ["The diamond plot is an histogram, where the axis are the rank of the tokens", "A perfect diagonal would be if all names were of the same rank for both years", "As you get further away from the diagonal on the right, you get the most popular boy babynames for 1930", "Finally we can put everything together, contrasting two different years in different ways"] as text, i}
		  <div class='step' class:active={currentStep === i}>
			<div class="step-content">
			  <p>{text}</p>
			</div>
		  </div>
		  {/each}
		</Scrolly>
	  </div>
	</section>
</main>
  
<style>
	:global(*) {
	  font-family: Inter;
	  -moz-osx-font-smoothing: grayscale;
	}
	
	main {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		text-align: center;
		margin: 0 auto;
		width: 100%;
	}

	h1 {
	  margin-top: 5rem;
	  font-size: 50px;
	  margin-bottom: 1rem;
	}
  
	h2 {
	  font-size: 20px;
	  margin-bottom: 3rem;
	  color: hsla(212, 10%, 53%, 1);
	}
  
	p:not(:last-of-type) {
	  margin-bottom: 1rem;
	}
  
	/* Scrollytelling stuff */
  
	.step {
	  height: 90vh;
	  opacity: 0.3;
	  transition: opacity 300ms ease;
  
	  display: flex;
	  justify-content: center;
	  place-items: center;
	}
  
	.step.active {
	  opacity: 1;
	}
  
	.step-content {
	  background-color: white;
	  border: 1px solid black;
	  padding: 0.75rem 1rem;
	  border-radius: 3px; 
	}
  
	section {
	  position: relative;
	}
  
	.steps {
	  position: relative;
	  z-index: 2;
	}
  
	.sticky {
	  position: sticky;
	  margin-top: 30px;
	  height: 90vh;
	  top: 5vh; /* (100vh - 90vh) / 2 */
	  width: 100%; /* Full width */
	  z-index: 1;
	  margin-bottom: 1rem;
	}
  
	.reference-step {
	  position: fixed;
	  bottom: 0;
	  right: 0;
	  padding: 1rem;
	}
  
	.initial-chart {
	  margin-bottom: 1rem;
	}
  
	/* Transition section */
  
	:global(rect) {
	  transition: 
	  x 700ms cubic-bezier(0.76, 0, 0.24, 1), 
	  y 700ms cubic-bezier(0.76, 0, 0.24, 1),
	  fill 700ms cubic-bezier(0.76, 0, 0.24, 1),
	  width 700ms cubic-bezier(0.76, 0, 0.24, 1),
	  height 700ms cubic-bezier(0.76, 0, 0.24, 1),
	  opacity 700ms cubic-bezier(0.76, 0, 0.24, 1);
	}
  
	:global(text) {
	  transition: 
		x 700ms cubic-bezier(0.76, 0, 0.24, 1), 
		y 700ms cubic-bezier(0.76, 0, 0.24, 1),
		opacity 700ms cubic-bezier(0.76, 0, 0.24, 1);
	}
  
	:global(tick) {
	  transition: 
		x 700ms cubic-bezier(0.76, 0, 0.24, 1), 
		y 700ms cubic-bezier(0.76, 0, 0.24, 1)
		opacity 700ms cubic-bezier(0.76, 0, 0.24, 1);
	}
  </style> -->