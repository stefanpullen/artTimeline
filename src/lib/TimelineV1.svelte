<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';
	import { draw, fade, fly, slide } from 'svelte/transition';
	import { scaleLinear, max, min, ascending } from 'd3';
	import { cubicOut, quadIn, quadOut, cubicIn, quadInOut, cubicInOut, linear } from 'svelte/easing';
	import { reverse } from 'svg-path-reverse';
	import { typewriter } from './typewriter';

	export let artdat;

	let svg, roughCanvas, path, pathArray;
	let pathObjects = [];
	let visible = false;
	function toggle() {
		visible = !visible;
	}

	$: roughness = 0.6;

	let height = 600;
	let width = 1080;
	const margin = { top: 100, right: 50, bottom: 100, left: 100 };

	const startDomain = artdat.map((d) => +d.start_date);
	const endDomain = artdat.map((d) => +d.end_date);
	const sizeDomain = artdat.map((d) => +d.duration);

	const xScale = scaleLinear()
		.domain([min(startDomain), max(endDomain)])
		.range([margin.left, width - margin.right]);

	const period = max(endDomain) - min(startDomain);
	const innerWidth = width - margin.left - margin.right;
	const sizeRange1 = (min(sizeDomain) / period) * innerWidth;
	const sizeRange2 = (max(sizeDomain) / period) * innerWidth;

	const sizeScale = scaleLinear()
		.domain([min(sizeDomain), max(sizeDomain)])
		.range([sizeRange1, sizeRange2]);

	onMount(() => {
		roughCanvas = rough.svg(svg);
		pathObjects = artdat.map((d, index) => {
			const startPoint = xScale(d.start_date) + sizeScale(d.duration) / 2;
			const path = roughCanvas.arc(
				startPoint,
				height / 2,
				sizeScale(d.duration),
				sizeScale(d.duration),
				index % 2 == 0 ? 0 : Math.PI,
				index % 2 == 0 ? Math.PI : Math.PI * 2,
				true,
				{
					stroke: 'black',
					fill: 'black',
					roughness: roughness,
					hachureGap: 5,
					hachureAngle: index % 2 == 0 ? 180 : -90
				}
			);

			const anotationLine = roughCanvas
				.line(
					xScale(d.start_date),
					height / 2,
					xScale(d.start_date),
					index % 2 == 0 ? height * 0.9 : height * 0.1,
					true,
					{
						stroke: 'black',
						fill: 'none',
						strokeWidth: 10,
						roughness: 100
					}
				)
				.children[0].getAttribute('d');

			return {
				id: index,
				name: d.Movement,
				anotationLine: anotationLine,
				start: d.start_date,
				end: d.end_date,
				path: path.children[0].getAttribute('d'),
				stroke:
					index % 2 == 0
						? reverse(path.children[1].getAttribute('d'))
						: path.children[1].getAttribute('d')
			};
		});
		console.log(pathObjects);
	});

	let delay = (i) => i * 50 + 2200;
	let delay2 = (i) => i * 150 + 2250;
</script>

<svg viewBox="0 0 {width} {height}" bind:this={svg} id="mysvg">
	{#if visible}
		{#each pathObjects.sort((a, b) => ascending(a.start, b.start)) as d, i}
			<path
				id="stroke"
				stroke="black"
				stroke-width="1"
				fill="none"
				d={d.stroke}
				in:draw={{ duration: 3000, delay: 150 * i, speed: 0.5, easing: cubicOut }}
				out:draw={{ duration: 3000, delay: 150 * i, speed: 0.5, easing: cubicIn }}
			/>
			<path
				id="annotations-lines"
				stroke="black"
				stroke-width="1"
				fill="none"
				d={d.anotationLine}
				in:draw={{ duration: 3000, delay: 150 * i, speed: 0.5, easing: cubicOut }}
				out:draw={{ duration: 3000, delay: 150 * i, speed: 0.5, easing: cubicIn }}
			/>
			<path
				id="filling"
				stroke="black"
				stroke-width="1"
				fill="none"
				d={d.path}
				transition:draw={{ duration: 3000, delay: delay2(i), speed: 2, easing: cubicInOut}}
			/>
			<text
				class="label"
				id="back-label"
				x={xScale(d.start)}
				y={i % 2 == 0 ? height * 0.9 : height * 0.1}
				text-anchor={i % 2 == 0 ? 'end' : 'start'}
				in:fly={{
					y: i % 2 == 0 ? height * -0.2 : height * 0.2,
					duration: 3000,
					delay: delay2(i),
					easing: cubicOut
				}}
				out:fly={{
					y: i % 2 == 0 ? height * -0.2 : height * 0.2,
					duration: 3000,
					delay: delay2(i),
					easing: cubicOut
				}}
			>
				{d.name}
			</text>
			<text
				class="label"
				x={xScale(d.start)}
				y={i % 2 == 0 ? height * 0.9 : height * 0.1}
				text-anchor={i % 2 == 0 ? 'end' : 'start'}
				transition:fly={{
					y: i % 2 == 0 ? height * -0.2 : height * 0.2,
					duration: 3000,
					delay: delay2(i),
					easing: cubicOut
				}}
				>{d.name}
			</text>

			<text
				class="label_year"
				x={xScale(d.start)}
				y={i % 2 == 0 ? height * 0.9 : height * 0.1}
				text-anchor={i % 2 == 0 ? 'end' : 'start'}
				transition:fly={{
					y: i % 2 == 0 ? height * -0.2 : height * 0.2,
					duration: 3000,
					delay: delay2(i),
					easing: cubicOut
				}}
				>{d.start}
			</text>
		{/each}
		<path
			d="M{margin.left} {height / 2} L{width - margin.right} {height / 2}"
			stroke="black"
			stroke-width="7"
			id="one"
			in:draw={{ duration: 2250, easing:cubicOut }}
			out:draw={{ duration: 2250, easing:cubicIn }}

		/>
	{/if}



	<text class="title" x={margin.left / 2} y={height/2} transition:typewriter> Modern Art Movements</text>

</svg>

<button left={margin.left} on:click={toggle} />
<div class="text" id='start'>1867</div>
<div class="text" id='end'>1975</div>


<style>
	svg {
		background: #eeeeee;
		position: relative;
		display: block;
		margin: 0 auto;
		width: 75%;
	}

	.label {
		font-family: Courier, monospace;
		font-weight: 1000;
		baseline-shift: 5;
		writing-mode: vertical-rl;
		text-orientation: upright;
		font-size: 0.6em;
		letter-spacing: -2px;
		text-transform: capitalize;
	}

	#back-label {
		stroke: #eeeeee;
		stroke-width: 2;
	}

	.label_year {
		font-family: Courier, monospace;
		font-weight: 1000;
		baseline-shift: -4;
		writing-mode: vertical-rl;
		text-orientation: upright;
		font-size: 0.6em;
		letter-spacing: -2px;
		text-transform: capitalize;
	}

	.title {
		text-anchor:middle;
		font-family: Courier, monospace;
		letter-spacing: 3px;
		font-weight: 1000;
		font-size: 1.1rem;
		writing-mode: vertical-rl;
		text-orientation: upright;
	}
	.text {
		font-family: Courier, monospace;
		font-weight: 1000;
		position: absolute;
		background-color: none;
	}
	#start {
		top: 47%;
		left: 20%;
		display: none;

	}

	#end {
		top: 47%;
		left: 80%;
		display: none;

	}

	button {
		font-family: Courier, monospace;
		font-weight: 1000;
		font-size: 2em;
		position: absolute;
		top: 49.9%;
		left: 18.4%;
		transform: translatey(-50%);
		display: block;
		cursor: pointer;
		transition: all 0.3s ease-in-out;
		width: 1rem;
		height: 1rem;
		border-radius: 2rem;
		background-color: black;
	}

	button:hover + #end {
		display: block;
	}
	button:hover + #start {
		display: block;
	}
	button:hover  {
		background-color: #eeeeee;
	}
	button:active {
		background-color:black;
	}
</style>
