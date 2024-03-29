<!-- 
  Generate Legends
  Based on: https://observablehq.com/@d3/color-legend
 -->
<script>
	import { onMount } from 'svelte';
	import { scaleLinear, scaleSequential, quantize, interpolate, interpolateRound } from 'd3';
	import { color } from '$data/variables.json';

	export let colorScale;
	export let title = '';
	export let tickSize = 6;
	export let width = 320;
	export let height = 44 + tickSize;
	export let margin = { top: 20, right: 20, bottom: 16 + tickSize, left: 20 };
	export let nTicks = width / 64;

	let x;
	let n;
	let type;
	let xlink;
	let barWidth = width - margin.left - margin.right;
	let barHeight = height - margin.top - margin.bottom;

	$: tickVals = colorScale.ticks(nTicks);
	$: xScale = scaleLinear().domain(colorScale.domain()).range([margin.left, barWidth]);

	const genRamp = (colorScale, n = 256) => {
		const canvas = document.createElement('canvas');
		canvas.width = n;
		canvas.height = 1;
		const context = canvas.getContext('2d');
		for (let i = 0; i < n; i++) {
			context.fillStyle = colorScale(i / (n - 1));
			context.fillRect(i, 0, 1, 1);
		}
		return canvas;
	};

	onMount(() => {
		// continuous
		if (colorScale.interpolate) {
			type = 'continuous';
			n = Math.min(colorScale.domain().length, colorScale.range().length);
			x = colorScale.copy().rangeRound(quantize(interpolate(margin.left, width - margin.right), n));
			xlink = genRamp(colorScale.copy().domain(quantize(interpolate(0, 1), n))).toDataURL();

			// sequential
		} else if (colorScale.interpolator) {
			type = 'sequential';
			x = Object.assign(
				colorScale.copy().interpolator(interpolateRound(margin.left, width - margin.right)),
				{
					range() {
						return [margin.left, width - margin.right];
					}
				}
			);
			xlink = genRamp(colorScale.interpolator()).toDataURL();
		}
	});
</script>

<svg viewBox={`0 0 ${width} ${height}`}>
	<!-- <rect x={0} y={0} {width} {height} fill="red" opacity={0.2} /> -->

	<!-- COLOR SCALE -->
	{#if ['continuous', 'sequential'].includes(type)}
		<image
			x={margin.left}
			y={margin.top}
			width={barWidth}
			height={barHeight}
			preserveAspectRatio="none"
			xlink:href={xlink}
		/>
	{/if}

	<!-- AXIS AND LABELS -->
	<g transform={`translate(0,${height - margin.bottom})`}>
		{#each tickVals as tick}
			<line
				class="x-axis"
				x1={xScale(tick)}
				x2={xScale(tick)}
				y1={-barHeight}
				y2={0}
				stroke={color.white}
				stroke-width={1}
			/>
			<text
				class="tick-label"
				x={xScale(tick)}
				y={5}
				text-anchor="middle"
				dominant-baseline="hanging">{tick}</text
			>
		{/each}

		<!-- Title -->
		<text class="title" x={margin.left} y={-barHeight - 6} text-anchor="start">{title}</text>
	</g>
</svg>

<style>
	svg {
		display: block;
		overflow: 'visible';
	}

	.tick-label {
		fill: var(--color-g5);
		font-size: 10px;
		font-weight: 500;
	}

	.title {
		text-transform: uppercase;
		font-size: 10px;
		font-weight: 500;
		fill: var(--color-g5);
	}
</style>
