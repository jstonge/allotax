<script>
    import * as d3 from "d3";

    import AxisX from './AxisX.svelte';
    import AxisY from './AxisY.svelte';
    import Grid from './Grid.svelte';
    import Contours from './Contours.svelte';

    let { diamond_count, diamond_dat, margin, DiamondInnerHeight, trueDiamondHeight, alpha, maxlog10, rtd, title } = $props();
    
    import { get_relevant_types, rin } from "./utils_helpers"
    
    // Wrangling data
    let relevant_types = $derived(get_relevant_types(diamond_dat))

    // Extracting constant
    let max_rank_raw     = $derived(d3.range(d3.max(diamond_count, d => d.x1)));
    let max_rank         = $derived(d3.max(diamond_dat, (d) => d.rank_L[1]));
    let rounded_max_rank = $derived(10**Math.ceil(Math.max(Math.log10(max_rank))));
    let ncells           = $derived(d3.max(max_rank_raw));
    let xyDomain         = $derived([1, rounded_max_rank]);

    // All the Scales!
    let linScale    = $derived(d3.scaleLinear().domain([0,ncells-1]).range([0, DiamondInnerHeight]));
    let wxy         = $derived(d3.scaleBand().domain(max_rank_raw).range([0, DiamondInnerHeight]));
    let logScale    = $derived(d3.scaleLog().domain(xyDomain).range([0, DiamondInnerHeight]).nice());
    let xy          = $derived(d3.scaleBand().domain(max_rank_raw).range([0, trueDiamondHeight]));
    
    let color_scale = d3.scaleSequentialLog().domain([rounded_max_rank, 1]).interpolator(d3.interpolateInferno);     
    
    // Background colors
    let blue_triangle = [[DiamondInnerHeight, DiamondInnerHeight], [0, 0], [0, DiamondInnerHeight]].join(" ")
    let grey_triangle = [[DiamondInnerHeight, DiamondInnerHeight], [0, 0], [DiamondInnerHeight, 0]].join(" ")
    
    function filter_labs(d, relevant_types) {
        return rin(relevant_types, d.types.split(",")).some((x) => x === true)
    }


</script>

<g class='diamond-chart' transform="translate(360, 0) scale (-1,1) rotate(45) translate({margin.inner/2}, {margin.inner/2})">
    
    <polygon points={blue_triangle} fill="#89CFF0" fill-opacity=0.2 stroke="black" stroke-width=0.5/>
    <polygon points={grey_triangle} fill="grey" fill-opacity=0.2 stroke="black" stroke-width=0.5/>
    
    <AxisX height={DiamondInnerHeight} scale={logScale} {title}/>
    <AxisY height={DiamondInnerHeight} scale={logScale} {title}/>
    <Grid  height={DiamondInnerHeight} {wxy} {ncells} scale={linScale}></Grid>

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

    {#each diamond_dat.filter(d => filter_labs(d, relevant_types)) as d}
        <g class="diamond-lab" 
        transform="
            scale(1,-1) 
            rotate(-90) 
            rotate(-45, {xy(d.x1)}, {xy(d.y1)}) 
            translate({d.which_sys === "right" ? xy(Math.sqrt(d.cos_dist))*1.5 : -xy(Math.sqrt(d.cos_dist))*1.5}, 0)
        ">
        <text
            x={xy(d.x1)}
            y={Number.isInteger(d.coord_on_diag) ? xy(d.y1) : xy(d.y1)-1}
            dy={20}
            font-size="10"
            text-anchor={d.x1 - d.y1 <= 0 ? "start" : "end"}
        >{d.types.split(",")[0]}</text>
        </g>
    {/each}

    <Contours {alpha} {maxlog10} {rtd} {DiamondInnerHeight}></Contours>
</g>
