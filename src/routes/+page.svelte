<script lang="ts">
    import { combElems, RTD, myDiamond, wordShift_dat } from 'allotaxonometer';
    
    import { test_elem_1, test_elem_2 } from './data/babynames';  
    
    import DiamondChart from './components/DiamondChart.svelte';
    import Wordshift from './components/WordshiftChart.svelte';
    
    // Global width and height for the dashboard
    const margin = { top: 300, bottom: 300 };
    margin.diamon
	let dashboardWidth = $state();
	let diamondWidth = $derived(dashboardWidth * 2/3);
	let barChartWidth = $derived(dashboardWidth - diamondWidth);
    let dashboardHeight = 925;

    let diamondHeight = $derived(diamondWidth - margin.top - margin.bottom);
    
    // Wrangling data
    let alpha = $state(0.33);
	let me    = $derived(combElems(test_elem_1, test_elem_2));
    let rtd   = $derived(RTD(me, alpha));
    let dat   = $derived(myDiamond(me, rtd));
    let diamond_count = $derived(dat.counts);
	let diamond_dat   = $derived(diamond_count.filter(d => d.types !== ""))
	let barData = $derived(wordShift_dat(me, dat).slice(0, 30));
    let title = ['Boys 1895', 'Boys 1930']
    

</script>

<div class="chart-container" bind:clientWidth={dashboardWidth}>
    
	<!-- width={dashboardWidth} -->
	<svg
		width={dashboardWidth}
		height={dashboardHeight}
		viewBox="0 0 {dashboardWidth} {dashboardHeight}"
		style:max-width="100%"
		style:min-width="100%"
		style:height="auto"
		>
        <DiamondChart {diamond_count} {diamond_dat} {margin} {diamondHeight} />
        <Wordshift {barData} {dashboardHeight} {dashboardWidth} {barChartWidth}/>
    </svg>
</div>


<style>
	.chart-container {
    	position: relative;
  	}
</style>
