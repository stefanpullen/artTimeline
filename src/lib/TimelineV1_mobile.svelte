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
	let counter = 0;
	function count() {
		counter += 1;
	}

	$: roughness = 0.6;

	let height = 1080;
	let width = 600;
	const margin = { top: 100, right: 50, bottom: 50, left: 25 };

	const startDomain = artdat.map((d) => +d.start_date);
	const endDomain = artdat.map((d) => +d.end_date);
	const sizeDomain = artdat.map((d) => +d.duration);

	const xScale = scaleLinear()
		.domain([min(startDomain), max(endDomain)])
		.range([margin.top, height - margin.bottom]);

	const period = max(endDomain) - min(startDomain);
	const innerHeight = height - margin.top - margin.bottom;
	const sizeRange1 = (min(sizeDomain) / period) * innerHeight;
	const sizeRange2 = (max(sizeDomain) / period) * innerHeight;

	const sizeScale = scaleLinear()
		.domain([min(sizeDomain), max(sizeDomain)])
		.range([sizeRange1, sizeRange2]);

	onMount(() => {
		roughCanvas = rough.svg(svg);
		pathObjects = artdat
			.map((d, index) => {
				const startPoint = xScale(d.start_date) + sizeScale(d.duration) / 2;
				const path = roughCanvas.arc(
					width / 2,
					startPoint,
					sizeScale(d.duration),
					sizeScale(d.duration),
					index % 2 == 0 ? Math.PI * 0.5 : -Math.PI / 2,
					index % 2 == 0 ? Math.PI * 1.5 : Math.PI / 2,
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
						width / 2,
						xScale(d.start_date),
						index % 2 == 0 ? width * 0.1 : width * 0.9,
						xScale(d.start_date),
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
			})
			.sort((a, b) => ascending(a.start, b.start));
	});

	let delay = (i) => i * 150 + 5250;
	let delay2 = (i) => i * 150 + 7500;
</script>

<svg viewBox="0 0 {width} {height}" bind:this={svg} id="mysvg">
	<text class="title" y={margin.top / 4} x={width / 2}> Modern Art Movements</text>
	{#if visible}
		<path
			d="M{width / 2} {margin.top} L{width / 2} {height - margin.bottom}"
			stroke="black"
			stroke-width="3"
			id="one"
			transition:draw={{ delay: 3000, duration: 1500, easing: cubicOut }}
		/>

		<text class="name" y={margin.top / 2} x={width / 2} transition:typewriter
			>A timeline by Stefan Pullen</text
		>
		<text
			class="text"
			id="start"
			y={margin.top}
			x={width / 2 + 12}
			transition:fly={{
				x: -15,
				duration: 1500,
				delay: 4000,
				easing: cubicOut
			}}
		>
			1867</text
		>
		<text
			class="text"
			id="end"
			y={height - margin.bottom}
			x={width / 2 - 12}
			transition:fly={{
				x: 10,
				duration: 1500,
				delay: 4000,
				easing: cubicOut
			}}
		>
			1975</text
		>

		{#each pathObjects as d, i}
			<path
				id="stroke"
				stroke="black"
				stroke-width="1"
				fill="none"
				d={d.stroke}
				in:draw={{ duration: 3000, delay: delay(i), speed: 0.5, easing: cubicOut }}
				out:draw={{ duration: 3000, delay: delay(i), speed: 0.5, easing: cubicIn }}
			/>
			<path
				id="annotations-lines"
				stroke="black"
				stroke-width="1"
				fill="none"
				d={d.anotationLine}
				in:draw={{ duration: 3000, delay: delay(i), speed: 0.5, easing: cubicOut }}
				out:draw={{ duration: 3000, delay: delay(i), speed: 0.5, easing: cubicIn }}
			/>
			<path
				id="filling"
				stroke="black"
				stroke-width="1"
				fill="none"
				d={d.path}
				transition:draw={{ duration: 3000, delay: delay2(i), speed: 2, easing: cubicInOut }}
			/>
			<text
				class="label"
				id="back-label"
				Y={xScale(d.start)}
				X={i % 2 == 0 ? width * 0.1 : width * 0.9}
				text-anchor={i % 2 == 0 ? 'start' : 'end'}
				transition:fly={{
					x: i % 2 == 0 ? width * -0.2 : width * 0.2,
					duration: 3000,
					delay: delay2(i),
					easing: cubicOut
				}}
			>
				{d.name}
			</text>
			<text
				class="label"
				Y={xScale(d.start)}
				X={i % 2 == 0 ? width * 0.1 : width * 0.9}
				text-anchor={i % 2 == 0 ? 'start' : 'end'}
				transition:fly={{
					x: i % 2 == 0 ? width * 0.2 : width * -0.2,
					duration: 3000,
					delay: delay2(i),
					easing: cubicOut
				}}
				>{d.name}
			</text>

			<text
				class="label_year"
				Y={xScale(d.start)}
				X={i % 2 == 0 ? width * 0.1 : width * 0.9}
				text-anchor={i % 2 == 0 ? 'start' : 'end'}
				transition:fly={{
					x: i % 2 == 0 ? width * 0.2 : width * -0.2,
					duration: 3000,
					delay: delay2(i),
					easing: cubicOut
				}}
				>{d.start}
			</text>
		{/each}
	{/if}
</svg>
<button top={margin.top} on:click={toggle} />

<!-- <div class="text" id="start" >1867</div>
<div class="text" id="end">1975</div> -->
<style>
	svg {
		background: #eeeeee;
		position: relative;
		display: block;
		margin: 0 auto;
		height: 90%;
	}

	.label {
		font-family: Courier, monospace;
		font-weight: 1000;
		baseline-shift: 2;
		font-size: 0.6em;
		text-transform: capitalize;
		letter-spacing: 3px;
	}

	#back-label {
		stroke: #eeeeee;
		stroke-width: 2;
	}

	.label_year {
		font-family: Courier, monospace;
		font-weight: 1000;
		baseline-shift: -8;
		font-size: 0.6em;
		text-transform: capitalize;
		letter-spacing: 3px;
	}

	.title {
		text-anchor: middle;
		font-family: Courier, monospace;
		font-weight: 1000;
		font-size: 1.4rem;
		letter-spacing: 10.5px;
	}
	.name {
		text-anchor: middle;
		font-family: Courier, monospace;
		font-weight: 1000;
		font-size: 1rem;
		letter-spacing: 6.5px;
	}
	.text {
		font-family: Courier, monospace;
		font-weight: 1000;
		position: absolute;
		background-color: none;
		font-size: 1rem;
		writing-mode: vertical-rl;
		text-orientation: upright;
	}

	#end {
		text-anchor: end;
	}

	button {
		position: absolute;
		left: 50%;
		top: 12.1%;
		transform: translatex(-50%);
		display: block;
		cursor: pointer;
		transition: all 0.3s ease-in-out;
		width: 0.5rem;
		height: 0.5rem;
		border-radius: 2rem;
		background-color: black;
	}

	button:hover {
		background-color: #eeeeee;
	}
	button:active {
		background-color: black;
	}
</style>
