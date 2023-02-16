<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';
	import { draw, fly } from 'svelte/transition';
	import { scaleLinear, max, min, ascending } from 'd3';
	import { cubicOut, cubicIn, cubicInOut } from 'svelte/easing';
	import { reverse } from 'svg-path-reverse';
	import { typewriter } from './typewriter';

	export let artdat;

	let svg, roughCanvas, startButton;
	let pathObjects = [];
	let visible = false;
	function toggle() {
		visible = !visible;
	}
	function handleKeyDown(event) {
		if (event.key === 'Enter') {
			toggle();
		}
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
		startButton = roughCanvas
			.circle(width / 2, height / 2, 75, true, {
				stroke: 'black',
				fill: 'black',
				roughness: roughness,
				hachureGap: 5
			})
			.children[0].getAttribute('d');
		pathObjects = artdat
			.map((d, index) => {
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
			})
			.sort((a, b) => ascending(a.start, b.start));
	});

	let delay = (i) => i * 150 + 5250;
	let delay2 = (i) => i * 150 + 7500;
</script>

<svg
	viewBox="0 0 {width} {height}"
	bind:this={svg}
	id="mysvg"
	on:click={toggle}
	on:keydown={handleKeyDown}
>
	<text class="title" x={margin.left / 4} y={height / 2}>Modern Art Movements</text>
	{#if !visible}
		<path
			d={startButton}
			stroke="black"
			stroke-width="3"
			fill="black"
			out:fly={{
				x: width / 2,
				duration: 1500,
				easing: cubicIn
			}}
			in:fly={{
				delay: 3000,
				x: width / 2,
				duration: 1500,
				easing: cubicOut
			}}
		/>
	{/if}

	{#if visible}
		<path
			d="M{margin.left} {height / 2} L{width - margin.right} {height / 2}"
			stroke="black"
			stroke-width="3"
			id="one"
			in:draw={{ delay: 3000, duration: 1500, easing: cubicOut }}
			out:draw={{ duration: 1500, easing: cubicIn }}
		/>
		<text class="name" x={margin.left / 2} y={height / 2} in:typewriter
			>A timeline by Stefan Pullen</text
		>
		<text
			class="text"
			id="start"
			x={margin.left}
			y={height / 2 -7}
			in:fly={{
				y: 10,
				duration: 1500,
				delay: 4000,
				easing: cubicOut
			}}
			out:fly={{
				y: 10,
				duration: 1500,
				easing: cubicIn
			}}
		>
			1867</text
		>
		<text
			class="text"
			id="end"
			x={width - margin.right}
			y={height / 2 + 16}
			in:fly={{
				y: -15,
				duration: 1500,
				delay: 4000,
				easing: cubicOut
			}}
			out:fly={{
				y: -15,
				duration: 1500,
				easing: cubicIn
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
				out:draw={{ duration: 3000, speed: 0.5, easing: cubicIn }}
			/>
			<path
				id="annotations-lines"
				stroke="black"
				stroke-width="1"
				fill="none"
				d={d.anotationLine}
				in:draw={{ duration: 3000, delay: delay(i), speed: 0.5, easing: cubicOut }}
				out:draw={{ duration: 3000, speed: 0.5, easing: cubicIn }}
			/>
			<path
				id="filling"
				stroke="black"
				stroke-width="1"
				fill="none"
				d={d.path}
				in:draw={{ duration: 3000, delay: delay2(i), speed: 2, easing: cubicInOut }}
				out:draw={{ duration: 3000, speed: 2, easing: cubicInOut }}
			/>
			<text
				class="label"
				id="back-label"
				x={xScale(d.start)}
				y={i % 2 == 0 ? height * 0.9 : height * 0.1}
				text-anchor={i % 2 == 0 ? 'end' : 'start'}
				in:fly={{
					y: i % 2 == 0 ? width * -0.2 : width * 0.2,
					duration: 1500,
					delay: delay2(i),
					easing: cubicOut
				}}
				out:fly={{
					y: i % 2 == 0 ? width * -0.2 : width * 0.2,
					duration: 1500,
					delay: 500,
					easing: cubicIn
				}}
			>
				{d.name}
			</text>
			<text
				class="label"
				x={xScale(d.start)}
				y={i % 2 == 0 ? height * 0.9 : height * 0.1}
				text-anchor={i % 2 == 0 ? 'end' : 'start'}
				in:fly={{
					y: i % 2 == 0 ? height * -0.2 : height * 0.2,
					duration: 1500,
					delay: delay2(i),
					easing: cubicOut
				}}
				out:fly={{
					y: i % 2 == 0 ? height * -0.2 : height *0.2,
					duration: 1500,
					delay: 500,
					easing: cubicIn
				}}
				>{d.name}
			</text>

			<text
				class="label_year"
				x={xScale(d.start)}
				y={i % 2 == 0 ? height * 0.9 : height * 0.1}
				text-anchor={i % 2 == 0 ? 'end' : 'start'}
				in:fly={{
					y: i % 2 == 0 ? height * -0.2 : height * 0.2,
					duration: 1500,
					delay: delay2(i),
					easing: cubicOut
				}}
				out:fly={{
					y: i % 2 == 0 ? height * -0.2 : height * 0.2,
					duration: 1500,
					delay: 500,
					easing: cubicIn
				}}
				>{d.start}
			</text>
		{/each}
	{/if}
</svg>

<style>
	svg {
		background: #eeeeee;
		position: relative;
		display: block;
		margin: 0 auto;
		width: 75%;
	}

	.label {
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
		font-weight: 1000;
		baseline-shift: -4;
		writing-mode: vertical-rl;
		text-orientation: upright;
		font-size: 0.6em;
		letter-spacing: -2px;
		text-transform: capitalize;
	}

	.title {
		text-anchor: middle;
		letter-spacing: 3px;
		font-weight: 1000;
		font-size: 1.1rem;
		writing-mode: vertical-rl;
		text-orientation: upright;
	}
	.name {
		text-anchor: middle;
		font-weight: 1000;
		font-size: 0.7rem;
		letter-spacing: 0;
		writing-mode: vertical-rl;
		text-orientation: upright;
	}
	
	.text {
		font-weight: 1000;
		position: absolute;
		background-color: none;
	}

	#end {
		text-anchor: end;
	}
</style>
