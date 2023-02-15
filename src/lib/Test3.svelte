<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';
	import { draw } from 'svelte/transition';
	import { quintIn, quintOut } from 'svelte/easing';
	let svg, root, roughCanvas, paths, pat;
	let pathObjects = [];
	let condition = false;
	let visible = true;

	onMount(() => {
		roughCanvas = rough.svg(svg);
		paths = roughCanvas
			.rectangle(10, 100, 400, 200, {
				stroke: 'none',
				strokeWidth: 3,
				fill: 'black',
				roughness: 4,
				hachureGap: 10,
				hachureAngle: 0
			})
			.children[0].getAttribute('d');

		pat = paths.split('M');
		pat.shift();

		pat.forEach(function (path, index) {
			pathObjects.push({
				id: index,
				path: 'M' + path
			});
		});
		console.log(pathObjects);
		pathObjects = pathObjects;
	});
	console.log(pathObjects);
</script>

<label>
	<input type="checkbox" bind:checked={visible} />
	visible
</label>

<svg bind:this={svg}>


	{#each pathObjects as d, i}
		<path
			class="test"
			d={d.path}
			transition:draw={{ duration: 50 * (i * 5) , easing: quintIn }}
			stroke="black"
			fill="none"
		/>
	{/each}
</svg>

<style>
	svg {
		width: 600px;
		height: 600px;
		background: white;
	}
</style>
