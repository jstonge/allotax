<script lang="ts">
    import { combElems, RTD, myDiamond, wordShift_dat } from 'allotaxonometer';
    
    import { test_elem_1, test_elem_2 } from './data/babynames';  
    
    import DiamondChart from './components/DiamondChart.svelte';
    
    // Global width and height for the dashboard
    const margin = { top: 175, diamond: 40, bottom: 175 };
    
    let width = $state(700);
    let height = 925;

    // let innerWidth = $derived(width - margin.top - margin.bottom);   
    let diamondHeight = $derived(width - margin.top - margin.bottom);
    
    // Wrangling data
    let alpha = $state(0.33);
	let me    = $derived(combElems(test_elem_1, test_elem_2));
    let rtd   = $derived(RTD(me, alpha));
    let dat   = $derived(myDiamond(me, rtd));
    let diamond_count = $derived(dat.counts);
	let diamond_dat   = $derived(diamond_count.filter(d => d.types !== ""))
    
    let title = ['Boys 1895', 'Boys 1930']
    
	
</script>

<div class="chart-container">
    
	<svg
		{width}
		{height}
		viewBox="0 0 {width} {height}"
		style:max-width="100%"
		style:min-width="70%"
		style:height="auto"
	>
        <DiamondChart {diamond_count} {diamond_dat} {margin} {diamondHeight} />
        <!-- <Wordshift {barData} {height} {width}/> -->
    </svg>
</div>


<style>
	.chart-container {
		width: 100%;
    	position: relative;
  	}
</style>
