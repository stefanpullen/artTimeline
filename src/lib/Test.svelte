<script>
	import rough from 'roughjs/bin/rough';
	import { scaleLinear, scaleBand, max, min } from 'd3';

	export let artdat;
	let odd = artdat.filter((v, i) => i % 2);
	let even = artdat.filter((v, i) => !(i % 2));
	let roughSvg;

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

    function randomIntFromInterval(min, max) { // min and max included 
  return Math.floor(Math.random() * (max - min + 1) + min)
}

	function drawSemiCircle(node, x, y, size1, size2, startAngle, endAngle, hachAngle) {
		const semiCircle = roughSvg.arc(x, y, size1, size2, startAngle, endAngle, true, {
			stroke: 'none',
			strokeWidth: 1,
			fill: 'black',
			roughness: 0.4,
			hachureGap: 2,
			hachureAngle: hachAngle // angle of hachure,
		});
		node.appendChild(semiCircle);
	}

	function action(node) {
		roughSvg = rough.svg(node);

    for (let i = 1; i < odd.length; i += 2) {
			drawSemiCircle(
				node,
				xScale(odd[i].start_date) + sizeScale(odd[i].duration) / 2,
				height / 2,
				sizeScale(odd[i].duration),
				sizeScale(odd[i].duration),
				Math.PI,
				2 * Math.PI,
				90
			)
    }
    for (let i = 0; i < odd.length; i += 2) {
			drawSemiCircle(
				node,
				xScale(odd[i].start_date) + sizeScale(odd[i].duration) / 2,
				height / 2,
				sizeScale(odd[i].duration),
				sizeScale(odd[i].duration),
				Math.PI,
				2 * Math.PI,
				180
			)
    }

    for (let i = 1; i < even.length; i += 2) {
			drawSemiCircle(
				node,
				xScale(even[i].start_date) + sizeScale(even[i].duration) / 2,
				height / 2,
				sizeScale(even[i].duration),
				sizeScale(even[i].duration),
				Math.PI,
				2 * Math.PI,
				90
			)
    }
    for (let i = 0; i < even.length; i += 2) {
			drawSemiCircle(
				node,
				xScale(even[i].start_date) + sizeScale(odd[i].duration) / 2,
				height / 2,
				sizeScale(even[i].duration),
				sizeScale(even[i].duration),
        0, Math.PI,
				180
			)
    }


	}
</script>

<svg viewBox="0 0 {width} {height}" use:action>
	<path
		d="M{margin.left} {height / 2} L{width - margin.right} {height / 2}"
		stroke="black"
		stroke-width="4"
		id="one"
	/>
</svg>
