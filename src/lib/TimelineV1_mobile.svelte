<script>
	import { onMount } from 'svelte';
	import rough from 'roughjs';
	import { draw, fly, fade } from 'svelte/transition';
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
		startButton = roughCanvas
			.circle(width / 2, height / 2, 150, true, {
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
	let delay2 = (i) => i * 10 + 8000;
</script>

<svg
	viewBox="0 0 {width} {height}"
	bind:this={svg}
	id="mysvg"
	on:click={toggle}
	on:keydown={handleKeyDown}
>
	<text class="title" y={margin.top / 4} x={width / 2}>Modern Art Movements</text>
	{#if !visible}
		<path
			d={startButton}
			id="start-button"
			stroke="black"
			stroke-width="1"
			fill="#eeeeee"
			in:fade={{ delay: 3000, duration: 1500, easing: cubicOut }}
			out:fade={{ duration: 500, easing: cubicIn }}
		/>
		<text
			class="name2"
			x={width / 2}
			y={height / 2}
			in:fade={{ delay: 3000, duration: 1500, easing: cubicOut }}
			out:fade={{ duration: 500, easing: cubicIn }}>Start</text
		>
	{/if}
	{#if visible}
		<path
			d="M{width / 2} {margin.top} L{width / 2} {height - margin.bottom}"
			stroke="black"
			stroke-width="3"
			id="one"
			in:draw={{ delay: 3000, duration: 1500, easing: cubicOut }}
			out:draw={{ duration: 1500, easing: cubicIn }}
		/>

		<text class="name" y={margin.top / 2} x={width / 2} in:typewriter
			>A timeline by Stefan Pullen</text
		>
		<text
			class="text"
			id="start"
			y={margin.top}
			x={width / 2 + 12}
			in:fly={{
				x: -15,
				duration: 1500,
				delay: 4000,
				easing: cubicOut
			}}
			out:fly={{
				x: -15,
				duration: 1500,
				easing: cubicIn
			}}
		>
			1867</text
		>
		<text
			class="text"
			id="end"
			y={height - margin.bottom}
			x={width / 2 - 12}
			in:fly={{
				x: 15,
				duration: 1500,
				delay: 4000,
				easing: cubicOut
			}}
			out:fly={{
				x: 15,
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
				Y={xScale(d.start)}
				X={i % 2 == 0 ? width * 0.1 : width * 0.9}
				text-anchor={i % 2 == 0 ? 'start' : 'end'}
				in:fly={{
					x: i % 2 == 0 ? width * -0.2 : width * 0.2,
					duration: 1500,
					delay: delay2(i),
					easing: cubicOut
				}}
				out:fly={{
					x: i % 2 == 0 ? width * -0.2 : width * 0.2,
					duration: 1500,
					delay: 500,
					easing: cubicIn
				}}
			>
				{d.name}
			</text>
			<text
				class="label"
				Y={xScale(d.start)}
				X={i % 2 == 0 ? width * 0.1 : width * 0.9}
				text-anchor={i % 2 == 0 ? 'start' : 'end'}
				in:fly={{
					x: i % 2 == 0 ? width * 0.2 : width * -0.2,
					duration: 1500,
					delay: delay2(i),
					easing: cubicOut
				}}
				out:fly={{
					x: i % 2 == 0 ? width * 0.2 : width * -0.2,
					duration: 1500,
					delay: 500,
					easing: cubicIn
				}}
				>{d.name}
			</text>

			<text
				class="label_year"
				Y={xScale(d.start)}
				X={i % 2 == 0 ? width * 0.1 : width * 0.9}
				text-anchor={i % 2 == 0 ? 'start' : 'end'}
				in:fly={{
					x: i % 2 == 0 ? width * 0.2 : width * -0.2,
					duration: 1500,
					delay: delay2(i),
					easing: cubicOut
				}}
				out:fly={{
					x: i % 2 == 0 ? width * 0.2 : width * -0.2,
					duration: 1500,
					delay: 500,
					easing: cubicIn
				}}
				>{d.start}
			</text>
		{/each}
	{/if}
</svg>
<!-- <button top={margin.top} on:click={toggle}  /> -->

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
		font-weight: 1000;
		baseline-shift: -8;
		font-size: 0.6em;
		text-transform: capitalize;
		letter-spacing: 3px;
	}

	.title {
		text-anchor: middle;
		font-weight: 1000;
		font-size: 1.4rem;
		letter-spacing: 10.5px;
	}
	.name {
		text-anchor: middle;
		font-weight: 1000;
		font-size: 1rem;
		letter-spacing: 6.5px;
	}

	.name2 {
		text-anchor: middle;
		/* baseline-shift: -4; */
		font-weight: 1000;
		font-size: 0.7rem;
		letter-spacing: 0;
		background: #ffffff;
		letter-spacing: 1px;
		font-weight: 1000;
		font-size: 1.1rem;
		pointer-events: none;
		writing-mode: vertical-rl;
		text-orientation: upright;
	}
	.text {
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
</style>
