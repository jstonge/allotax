<script lang="ts">
    import Papa from 'papaparse';

    import { combElems, RTD, myDiamond, wordShift_dat } from 'allotaxonometer';
    
    import Diamond from './components/DiamondChart.svelte';
    import Wordshift from './components/WordshiftChart.svelte';
	import Legend from './components/Legend.svelte'
	import DivergingBarChart from './components/DivergingBarChart.svelte'
    
    import { test_elem_1, test_elem_2 } from './data/babynames';  

    let sys1 = $state(test_elem_1);
    let sys2 = $state(test_elem_2);
    
    let selected_sys1 = $state();
	let selected_sys2 = $state();

    let files = $state();

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

    // Global width and height for the dashboard
    const margin = { inner: 160, diamond: 40 };
    
    let DashboardHeight = 815;
    let DashboardWidth = $state(1200);
    
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

</script>

<aside>
    <p>Upload files:</p>
	
    <input bind:files id="many" multiple type="file" />
	
	{#if files}
			<p>select system 1</p>
			<div class="wrapper">
				<select onclick={load_sys1} bind:value={selected_sys1}>
					{#each files as file}
						<option value={file}>{file.name}</option>
					{/each}
				</select>		
			</div>
			<p>select system 2</p>
			<div class="wrapper">
				<select onclick={load_sys2} bind:value={selected_sys2}>
					{#each files as file}
						<option value={file}>{file.name}</option>
					{/each}
				</select>		
			</div>
	{/if}
</aside>

<main>
    <div class="dashboard" bind:clientWidth={DashboardWidth}>
        <svg id="mysvg" height={DashboardHeight} width={DashboardWidth}>
            <Diamond {diamond_count} {diamond_dat} {DiamondInnerHeight} {margin} {trueDiamondHeight} />
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
		margin-top: 100px;
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
			z-index: 1; /* Stay on top */
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

</style>
