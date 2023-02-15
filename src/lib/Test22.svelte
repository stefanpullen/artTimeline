<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';
	import { draw } from 'svelte/transition';
	import { scaleLinear, scaleBand, max, min } from 'd3';

	export let artdat;
	let odd = artdat.filter((v, i) => i % 2);
	let even = artdat.filter((v, i) => !(i % 2));
	let svg, root, roughCanvas, paths;
	let condition = false;

	let height = 500;
	let width = 1080;
	const margin = { top: 50, right: 50, bottom: 50, left: 50 };

	$: startDomain = artdat.map((d) => +d.start_date);
	$: endDomain = artdat.map((d) => +d.end_date);
	$: sizeDomain = artdat.map((d) => +d.duration);

	$: xScale = scaleLinear()
		.domain([min(startDomain), max(endDomain)])
		.range([margin.left, width - margin.right]);

	$: period = max(endDomain) - min(startDomain);
	$: innerWidth = width - margin.left - margin.right;
	$: sizeRange1 = (min(sizeDomain) / period) * innerWidth;
	$: sizeRange2 = (max(sizeDomain) / period) * innerWidth;

	$: sizeScale = scaleLinear()
		.domain([min(sizeDomain), max(sizeDomain)])
		.range([sizeRange1, sizeRange2]);

	onMount(() => {
		roughCanvas = rough.svg(svg);
		paths = roughCanvas
			.arc(100, 100, 180, 180, 0, Math.PI, true, {
				stroke: 'none',
				strokeWidth: 1,
				fill: 'black',
				roughness: 0.4,
				hachureGap: 2,
				hachureAngle: 45 // angle of hachure,
			})
			.children[0].getAttribute('d');
		console.log(paths);
	});
</script>

<button on:click={() => (condition = !condition)}> show svg </button>

<svg bind:this={svg}>
	{#if condition}
		<path stroke="black" d={paths} transition:draw={{ duration: 1500 }} />
	{/if}
</svg>

<style>
	svg {
		width: 600px;
		height: 600px;
		background: lightgrey;
	}
</style>
