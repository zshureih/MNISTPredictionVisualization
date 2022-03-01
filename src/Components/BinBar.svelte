<script>
    import { onMount } from "svelte";
    export let data;
    export let offset;
	import { select, selectAll } from "d3-selection" 
    import { hoveredPrediction, labelFilter, predictionFilter } from "../stores.js"

    console.log($labelFilter)
    console.log($predictionFilter)

    let colors = [
        '#114B5Fff', // 
		'#1A936Fff', // 
		'#88D498ff', // 
		'#C6DABFff', //  
		'#F3E9D2ff', //  
		'#DA4167ff', //  
		'#F78764ff', //  
		'#9D4E19ff', //  
		'#FCCA46ff', //  
		'#97C8EBff', //
	]
	let colorScale = function (label) {
		return colors[label]
	}

    let boxLength = 15
    
    export let instances = [];

    let locations = [];

    onMount(async () => {
        // Let's just unload all the data into a list for now
        let bottom_line = document.getElementById('bottom_line' + offset)
        let rect = bottom_line.getBoundingClientRect()
        let bar_width = rect.width
        let bin_width = bar_width / 10
        let number_boxes = Math.floor(bin_width / boxLength) - 1
        
        let starting_x = 100
        let starting_y = 100

        let i = 0
        data.bins.forEach(bin => {
            let bin_left = ((i * (bin_width)) + starting_x)
            let bin_bottom = starting_y

            let j = 0
            bin.instances.forEach(instance => {
                let box_x = bin_left + (j * boxLength)
                let box_y = bin_bottom + (boxLength / 2)
                locations.push([box_x, box_y])
                // this is the last box in this row
                if (j % number_boxes == 0 && j != 0){
                    bin_bottom -= boxLength
                    // increase the height of the next box, reset j
                    j = -1
                }
                j += 1
                instances.push(instance)
            });
            i += 1
        });
        instances = instances
        locations = locations
       
        // calculate columns per bin
        // each bin is 8.5%, starting from 10% (10, 18.5, 27, 35.5 and so on)
        
	});
</script>

<svg width=100%>
    {#if instances.length !== 0}
        <!-- text on the left -->
        <text x=0 y=50% stroke="black" font-size=14>Class</text>
        <rect x=40 y=40% width={2*boxLength} height={2*boxLength} style="fill:{colorScale(offset)}"> </rect>
        <image x=40 y=40% width={2*boxLength} height={2*boxLength} href='static/images/{instances[0].filename}'></image>

        <!-- 'Labeled as' button -->
        <text
            x=0 
            y=70%
            stroke="grey" 
            font-size=10
            on:click={() => {
                if ($labelFilter !== offset){
                    $labelFilter = offset
                } else{
                    $labelFilter = ''
                }
                console.log($predictionFilter, $labelFilter)

            }}
        >
            Labeled as {offset}
        </text>


        <!-- 'Predicted as' button -->
        <text 
            x=0 
            y=80% 
            stroke="grey" 
            font-size=10
            on:click={() => {
                if ($predictionFilter !== offset){
                    $predictionFilter = offset
                } else{
                    $predictionFilter = ''
                }
                console.log($predictionFilter, $labelFilter)
            }}
        >
            Predicted as {offset}
        </text>
        

        <!-- Bars -->
        {#if $hoveredPrediction !== null}
            <rect
                x=10%
                y=0
                height=100%
                width={Math.max($hoveredPrediction['predicted_scores'][offset] * 85)}%
                style="opacity: 0.5;fill:{colorScale(offset)}"
            >
            </rect>
        {/if}
        
        <!-- Binning system -->
        {#each locations as location, i}
            <rect
                class="
                    {
                        $hoveredPrediction === null ? 'noFilter' 
                            : ($hoveredPrediction['id'] === instances[i]["id"] ? 'highlighted' : 'filtered')
                    }                   
                    
                    {
                        $labelFilter === '' ? 'noFilter' 
                            : ($labelFilter === offset ? 'highlighted' : 'filtered')
                    }

                    {
                        $predictionFilter === '' ? 'noFilter' 
                            : ($predictionFilter === offset ? 'highlighted' : 'filtered')
                    }
                "
                x={location[0]}
                y={location[1]}
                width={boxLength}
                height={boxLength}
                style="fill:{colorScale(instances[i].true_label)};stroke:{colorScale(instances[i].predicted_label)}"
            >
            </rect>

            <image
                x={location[0]}
                y={location[1]}
                width={boxLength}
                height={boxLength}
                href='static/images/{instances[i].filename}'
                on:mouseenter={() => {
                    $hoveredPrediction = instances[i]
                }}
                on:mouseleave={() => {
                    $hoveredPrediction = null
                }}
            >
            </image>
        {/each}
        
    {/if}
    <!-- line at the bottom -->
    <line id={"bottom_line" + offset} x1=10% x2=95% y1=100% y2=100% stroke="black"></line>
    {#each [0.0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0] as tick, i}
        <line stroke="black" x1="{(i*8.5) + 10}%" x2="{(i*8.5) + 10}%" y1=98% y2=100%></line>
    {/each}
</svg>

<style>
    rect.noFilter {
		opacity: 0.75;
	}
	rect.highlighted {
		stroke-width: 1%;
		opacity: 1.0;
	}
	rect.filtered {
		opacity: 0.33;
	}
</style>