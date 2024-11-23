<script lang="ts">
	import * as d3 from "d3";
    import Papa from 'papaparse';

    import { combElems, RTD, myDiamond, wordShift_dat } from 'allotaxonometer';
    
    import Diamond from './components/DiamondChart.svelte';
    import Wordshift from './components/WordshiftChart.svelte';
	import Legend from './components/Legend.svelte'
	import DivergingBarChart from './components/DivergingBarChart.svelte'
    import MathJax from './MathJax.svelte';

	import { downloadChart, get_relevant_types } from "./components/utils_helpers"

    import { test_elem_1, test_elem_2 } from './data/babynames';  

    let sys1 = $state(test_elem_1);
    let sys2 = $state(test_elem_2);
    
    let files = $state();

    let selected_sys1 = $state();
	let selected_sys2 = $state();

    async function load_sys1(e: any) {
		Papa.parse(e.target.__value, {
			header: true, 
			complete: function(r: any) { sys1 = r.data; }
		});
	}

	async function load_sys2(e: any) {
		Papa.parse(e.target.__value, {
			header: true, 
			complete: function(r: any) { sys2 = r.data; }
		});
	}

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
    let alpha = $state(0.33);
	
	let me = $derived(combElems(sys1, sys2));
    let rtd = $derived(RTD(me, alpha));
    let dat = $derived(myDiamond(me, rtd));
    let diamond_count = $derived(dat.counts);
	let diamond_dat = $derived(diamond_count.filter(d => d.types !== ""))
    
	let barData = $derived(wordShift_dat(me, dat).slice(0, 30));

	let maxlog10 = $derived(Math.ceil(d3.max([Math.log10(d3.max(me[0].ranks)), Math.log10(d3.max(me[1].ranks))])))
	
	let delta_sum = $derived(d3.sum(dat.deltas.map(d => +d)).toFixed(3));
	let math = $derived(`$$D_{\\alpha}^R (\\Omega_1 || \\Omega_2 = ${delta_sum})$$\n$$\\propto \\sum_\\tau | \\frac{1}{r_{\\tau,1}^{${alpha}}} - \\frac{1}{r_{\\tau,2}^{${alpha}}} |$$`);
	let title = $derived(['Boys 1895', 'Boys 1930'])
	
	const alphas = d3.range(0,18).map(v => +(v/12).toFixed(2)).concat([1, 2, 5, Infinity])
	
</script>

<aside>
	<div class="alpha-slider">
		<p>Choose alpha:</p>
		<!-- here the value are the index of our alphas array, not the actual values-->
		<input 
			type="range"
			min="0"
			max="{alphas.length-1}"
			oninput={(e) => {alpha = alphas[e.target.value]}} 
			list=mapsettings
			>
		{#each alphas as ax}
			<datalist id="mapsetting"> <option>{ax}</option> </datalist>
		{/each}
		<p>α={alpha}</p>
	</div>
	<div>
		<p>Rank turbulence: </p>
		<MathJax {math}/>
	</div>
    <p>Upload files:</p>
	
    <input bind:files id="many" multiple type="file" />
	
	{#if files}
			<p>select system 1</p>
			<div class="wrapper">
				<select onclick={load_sys1} bind:value={selected_sys1} multiple=true>
					{#each files as file}
						<option value={file}>{file.name}</option>
					{/each}
				</select>		
			</div>
			<p>select system 2</p>
			<div class="wrapper">
				<select onclick={load_sys2} bind:value={selected_sys2} multiple=true>
					{#each files as file}
						<option value={file}>{file.name}</option>
					{/each}
				</select>		
			</div>
	{/if}
	<div class="download-button">
		<button onclick={downloadChart}>Download Image</button>
	</div>
</aside>


<main>
    <div class="dashboard">
        <svg id="mysvg" height={DashboardHeight} width={DashboardWidth}>
            <Diamond {diamond_count} {diamond_dat} {DiamondInnerHeight} {margin} {trueDiamondHeight} {alpha} {maxlog10} {rtd}/>
            <Wordshift {barData} {DashboardHeight} {DashboardWidth}/>
			<DivergingBarChart {test_elem_1} {test_elem_2} {DiamondHeight} {DiamondWidth} />
			<Legend {diamond_dat} {DiamondHeight}/> 
        </svg>
    </div>
	
</main>


<style>
	main {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		text-align: center;
		margin: 0 auto;
		margin-top: -60px;
		margin-left: 150px;
		width: 100%;
	}

	@media (min-width: 800px) {


	.dashboard {
    	position: relative;
  	}

	  aside {
			align-self: start;
			height: 100%; /* Full-height: remove this if you want "auto" height */
			width: 250px; /* Set the width of the sidebar */
			position: fixed; /* Fixed Sidebar (stay in place on scroll) */
			z-index: 0; /* Stay on top */
			top: 0; /* Stay at the top */
			left: 0;
			background-color: #ffffff83; /* Black */
			overflow-x: hidden; /* Disable horizontal scroll */
			padding-top: 20px;
			padding-left: 5px;
		}
	}

	.wrapper{
		display: flex;
		align-items: center;
		justify-content: center;
		padding-top: 10px;
		max-height: 120px;
   		overflow-y: auto;
	}

	.download-button {
		margin-top: 20px;
	}

</style>
