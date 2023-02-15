<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';
	import { draw } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';
	import { Path } from 'd3';

	let svg, root, roughCanvas, paths;
	let condition = false;

	onMount(() => {
		roughCanvas = rough.svg(svg);
		paths = roughCanvas
			.rectangle(10, 10, 100, 100, { seed: 12, fill: 'red', fillWeight: 0.5 })
			.children[0].getAttribute('d');
		console.log(paths);
	});
</script>

<button on:click={() => (condition = !condition)}> show svg </button>

<svg bind:this={svg}>
	{#if condition}
		<!-- <g bind:this={root} transition:draw={{ duration: 1500, easing: quintOut }} /> -->
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
