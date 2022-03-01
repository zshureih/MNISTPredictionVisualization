

<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import BinBar from "./Components/BinBar.svelte"
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";
	import { extent, max } from "d3-array";
    import { hoveredPrediction, labelFilter, predictionFilter } from "./stores.js"

	export let instances = [];
	export let binsByClasses = [];

	const numClasses = 10;
	const numBins = 10;

	let boxLength = 15;
	let y;
	
	$hoveredPrediction = null

	let xScale = scaleLinear()
	let yScale = scaleLinear()
	let ticks = [
		"0.0",
		"0.1",
		"0.2",
		"0.3",
		"0.4",
		"0.5",
		"0.6",
		"0.7",
		"0.8",
		"0.9",
		"1.0"
	]

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

	function parseScroll() {
		console.log(y)
	}

	onMount(async () => {
		const fetched = await fetch("static/prediction_results.json");
		instances = (await fetched.json()).test_instances;

		xScale = scaleLinear()
			.domain(extent(instances, function (d) {return d.projection[0]}))
			.range([5, 400]);

		yScale = scaleLinear()
			.domain(extent(instances, function (d) {return d.projection[1]}))
			.range([5, 295]);

		// Transform data
		for (let k = 0; k < numClasses; ++k) {
			let binsForClass = [];
			for (let b = 0; b < numBins; ++b) {
				binsForClass.push({"class": k, "binNo": b, "instances": []});
			}
			binsByClasses.push({"class": k, "bins": binsForClass});
		}

		// sort the mnist data into bins
		instances.forEach(instance => {
			instance.selected = false
			instance.filtered = false

			let true_label = instance.true_label
			let pred_conf = instance.predicted_scores[true_label]
			let bin = Math.floor(pred_conf * 10)
			binsByClasses[instance.true_label].bins[Math.min(bin, 9)].instances.push(instance)
		});
		binsByClasses = binsByClasses
	});
	

</script>

<main>
	<h1>Visual Analytics HW 3</h1>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="projection-view" class="view-panel">
				<div class="view-title">Projection View</div>
				<svg width="600" height="300">
					{#each instances as instance, i}
						<circle 
							class="
								{$hoveredPrediction == null ? 'noFilter' : (
									$hoveredPrediction['id'] == instance["id"] ? 'highlighted' : 'filtered'
								)}

								{
									$labelFilter === '' ? 'noFilter' 
										: ($labelFilter === instance.true_label ? 'highlighted' : 'filtered')
								}

								{
									$predictionFilter === '' ? 'noFilter' 
										: ($predictionFilter === instance.predicted_label ? 'highlighted' : 'filtered')
								}
							"
							r=5
							cx={xScale(instance.projection[0])} 
							cy={yScale(instance.projection[1])} 
							fill={colorScale(instance.true_label)} 
							stroke={colorScale(instance.predicted_label)}
							on:mouseenter={() => {
								$hoveredPrediction = instance
							}}
							on:mouseleave={() => {
								$hoveredPrediction = null
							}}
						/>
					{/each }
				</svg>
			</div>

			<div id="selected-image-view" class="view-panel">
				<div class="view-title">Selected Image</div>
				<div id="selected-image-view-content">
					<svg>
						{#if $hoveredPrediction != null}
							<image 
								x=0 
								y=0 
								width={10*boxLength} 
								height={10*boxLength} 
								href='static/images/{$hoveredPrediction.filename}'>
							</image>
							<text x=60% y=10%>
								ID: {$hoveredPrediction['id']}
							</text>
							<text x=60% y=20%>
								Labeled as: {$hoveredPrediction['true_label']}
							</text>
							<text x=60% y=30%>
								Predicted as: {$hoveredPrediction['predicted_label']} 
							</text>
							<text x=60% y=40%>
								Confidence: {$hoveredPrediction['predicted_scores'][$hoveredPrediction['predicted_label']]}
							</text>
						{/if}
					</svg>
				</div>
			</div>
		</div>

		<div id="main-section" style="width: 1000px;">
			<div id="score-distributions-view" class="view-panel">
				<div class="view-title">Score Distributions</div>
					<!-- top text bar -->
					<svg left=10% width=95% height=2%>
						{#each ticks as tick, i}
							<text x="{(i*8.7) + 10}%" y=15>{tick}</text>
						{/each}
						<line x1="10%" x2="100%" y1=20 y2=20 stroke="black"></line>
					</svg>
					<!-- class rows -->
					<div class="bins" bind:this={y}>
						{#each binsByClasses as bin, i}	
							<BinBar data={bin} offset={i}></BinBar>
						{/each }
					</div>
			</div>
		</div>
	</div>
</main>

<style>
	h1 {
		font-size: 1.3rem;
		font-weight: 500;
		margin-top: 0;
	}
	#container {
		display: flex;
	}
	#sidebar, #main-section {
		display: flex;
		flex-direction: column;
	}
	.view-panel {
		border: 2px solid #eee;
		margin-bottom: 15px;
		margin-right: 15px;
	}
	.view-title {
		background-color: #f3f3f3;
		font-size: 1.0rem;
		margin-bottom: 8px;
		padding: 3px 8px 5px 12px;
		max-height: 10%
	}
	#selected-image-view-content {
		height: 300px;
		width: 100%
		/* padding: 15px; */
	}
	svg {
		margin: 5px;
	}
	circle.noFilter {
		opacity: 0.75;
	}
	circle.highlighted {
		stroke-width: 1%;
		opacity: 1.0;
		r: 10;
	}
	circle.filtered {
		opacity: 0.33;
		r: 3;
	}
	.bins {
		height: 200vh;
	}
</style>