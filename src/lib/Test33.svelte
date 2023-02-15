<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';
	import { draw } from 'svelte/transition';
	import { scaleLinear, scaleBand, max, min } from 'd3';
	import { quintIn, quintOut } from 'svelte/easing';

	export let artdat;
	let odd = artdat.filter((v, i) => i % 2);
	let even = artdat.filter((v, i) => !(i % 2));
	let svg, root, roughCanvas, path, pathArray;
	let pathObjects = [];
	let visible = true;

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
		path = roughCanvas
			.arc(100, 100, 180, 180, 0, Math.PI, true, {
				stroke: 'none',
				strokeWidth: 1,
				fill: 'black',
				roughness: 0.4,
				hachureGap: 5,
				hachureAngle: 180 // angle of hachure,
			})
			.children[0].getAttribute('d');

		pathArray = path.split('M');
		pathArray.shift();

		pathArray.forEach(function (path, index) {
			pathObjects.push({
				id: index,
				path: 'M' + path
			});
		});
		console.log(pathObjects);
		pathObjects = pathObjects;
	});
</script>

<label>
	<input type="checkbox" bind:checked={visible} />
	visible
</label>

<svg bind:this={svg}>
	{#if visible}
		{#each pathObjects as d, i}
			<path
				class="test"
				d={d.path}
				transition:draw={{ duration: 5 * (i), easing: quintIn }}
				stroke="black"
				fill="none"
			/>
		{/each}
	{/if}
</svg>

<style>
	svg {
		width: 600px;
		height: 600px;
		background: lightgrey;
	}
</style>
