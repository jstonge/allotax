<script>
    import * as d3 from "d3";

    import AxisX from './AxisX.svelte';
    import AxisY from './AxisY.svelte';
    import Grid from './Grid.svelte';
    import Legend from './Legend.svelte';

    let { diamond_count, diamond_dat, margin, diamondHeight } = $props();
    
    import { get_relevant_types, rin } from "./utils_helpers"
    
    // Wrangling data
    let title = ['Boys 1895', 'Boys 1930']
    let relevant_types = $derived(get_relevant_types(diamond_dat))

    // Extracting constant
    let max_rank_raw     = $derived(d3.range(d3.max(diamond_count, d => d.x1)));
    let max_rank         = $derived(d3.max(diamond_dat, (d) => d.rank_L[1]));
    let rounded_max_rank = $derived(10**Math.ceil(Math.max(Math.log10(max_rank))));
    let ncells           = $derived(d3.max(max_rank_raw));
    let xyDomain         = $derived([1, rounded_max_rank]);

    // All the Scales!
    let linScale    = $derived(d3.scaleLinear().domain([0,ncells-1]).range([0, diamondHeight]));
    let wxy         = $derived(d3.scaleBand().domain(max_rank_raw).range([0, diamondHeight]));
    let logScale    = $derived(d3.scaleLog().domain(xyDomain).range([0, diamondHeight]).nice());
    let xy          = $derived(d3.scaleBand().domain(max_rank_raw).range([0, diamondHeight]));
    
    let color_scale = d3.scaleSequentialLog().domain([rounded_max_rank, 1]).interpolator(d3.interpolateInferno);     
    
    // Background colors
    let blue_triangle = $derived([[diamondHeight, diamondHeight], [0, 0], [0, diamondHeight]].join(" "))
    let grey_triangle = $derived([[diamondHeight, diamondHeight], [0, 0], [diamondHeight, 0]].join(" "))

    function filter_labs(d, relevant_types) {
        return rin(relevant_types, d.types.split(",")).some((x) => x === true)
    }


</script>

    <!-- <g class='inner-chart' > -->
    <g class='inner-chart' transform="translate(460, 0)  scale (-1,1)  rotate(45) translate(50,50)">
                        
        <polygon points={blue_triangle} fill="#89CFF0" fill-opacity=0.2 stroke="black" stroke-width=0.5/>
        <polygon points={grey_triangle} fill="grey" fill-opacity=0.2 stroke="black" stroke-width=0.5/>
        
        <AxisX width={diamondHeight} scale={logScale} {title}/>
        <AxisY height={diamondHeight} scale={logScale} {title}/>
        <Grid  height={diamondHeight} {wxy} {ncells} scale={linScale}></Grid>

        {#each diamond_dat as d}
            <rect
                x={xy(d.x1)}
                y={xy(d.y1)}
                width={xy.bandwidth()}
                height={xy.bandwidth()}
                fill={color_scale(d.value)}
                opacity={d.value === null ? 0 : 1.}
                stroke="black"
                stroke-width=0.2
            />
        {/each}
        
        <Legend {diamond_dat} DiamondHeight={diamondHeight}/> 
    </g>

<style>
    .inner-chart {
        position: relative;
    }
</style>
