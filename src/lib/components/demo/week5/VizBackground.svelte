<script>
	import * as d3 from 'd3';
	import { fade } from 'svelte/transition';
	import csv from '$lib/data/acc_species_trend.csv?raw';

	// Parse the CSV once and coerce year/count to numbers.
	const dataset = d3.csvParse(csv, (d) => ({
		year: +d.year,
		section: d.section,
		metric: d.metric,
		species: d.species,
		count: +d.count
	}));

	let { index = 0, count = 1, data = dataset } = $props();

	// One entry per scroll step. Each step selects a (section, metric) slice
	// of the shelter data. `highlight` emphasizes a single species.
	const steps = [
		{ section: 'Intake', metric: 'Total', highlight: null, annotation: null },
		{ section: 'Intake', metric: 'Stray/At Large', highlight: null, annotation: null },
		{ section: 'Live Outcome', metric: 'Adoption', highlight: null, annotation: null },
		{ section: 'Live Outcome', metric: 'Adoption', highlight: 'Cats', annotation: null },
		{
			section: 'Live Outcome',
			metric: 'Adoption',
			highlight: 'Dogs',
			annotation: { species: 'Dogs', year: 2024, text: 'Dog adoptions dip in 2024' }
		}
	];

	const margin = { top: 30, right: 30, bottom: 40, left: 56 };
	const colors = { Dogs: '#457b9d', Cats: '#e76f51', Rabbits: '#2a9d8f', 'Guinea Pigs': '#e9c46a' };

	let width = $state(700);
	let height = $state(460);

	// clamp so the last step's state holds once the reader scrolls past it
	const step = $derived(steps[Math.min(index, steps.length - 1)]);
	const section = $derived(step.section);
	const metric = $derived(step.metric);
	const highlight = $derived(step.highlight);
	const annotation = $derived(step.annotation);

	// rows for the current section/metric slice
	const view = $derived(data.filter((d) => d.section === section && d.metric === metric));

	const years = $derived([...new Set(view.map((d) => d.year))].sort((a, b) => a - b));
	const species = $derived([...new Set(data.map((d) => d.species))]);

	const xScale = $derived(
		d3
			.scalePoint()
			.domain(years)
			.range([margin.left, width - margin.right])
	);

	const yScale = $derived(
		d3
			.scaleLinear()
			.domain([0, d3.max(view, (d) => d.count) ?? 0])
			.nice()
			.range([height - margin.bottom, margin.top])
	);

	const line = $derived(
		d3
			.line()
			.x((d) => xScale(d.year))
			.y((d) => yScale(d.count))
	);

	function seriesFor(sp) {
		return view.filter((d) => d.species === sp).sort((a, b) => a.year - b.year);
	}

	const annotationPoint = $derived.by(() => {
		if (!annotation) return null;
		const row = view.find((d) => d.species === annotation.species && d.year === annotation.year);
		if (!row) return null;
		return { x: xScale(row.year), y: yScale(row.count) };
	});
</script>

<div class="background">
	<div class="heading">
		<h2>{section} · {metric}</h2>
		<p>Animal shelter counts by species</p>
	</div>
	<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
		<svg viewBox="0 0 {width} {height}" preserveAspectRatio="none">
			<!-- y axis -->
			{#each yScale.ticks(5) as tick (`${section}-${metric}-${tick}`)}
				<g class="tick" transform="translate(0,{yScale(tick)})" transition:fade={{ duration: 250 }}>
					<line x1={margin.left} x2={width - margin.right} />
					<text x={margin.left - 8} text-anchor="end" dominant-baseline="middle">{tick}</text>
				</g>
			{/each}

			<!-- x axis -->
			{#each years as year (`${section}-${metric}-${year}`)}
				<text
					x={xScale(year)}
					y={height - margin.bottom + 22}
					text-anchor="middle"
					transition:fade={{ duration: 250 }}
				>
					{year}
				</text>
			{/each}

			<!-- one line per species -->
			{#each species as sp (sp)}
				<path
					d={line(seriesFor(sp))}
					fill="none"
					stroke={colors[sp]}
					stroke-width={highlight === sp ? 4 : 2}
					opacity={highlight && highlight !== sp ? 0.15 : 1}
				/>
			{/each}

			<!-- annotation, only rendered when the current step has one -->
			{#if annotationPoint}
				<g class="annotation" transform="translate({annotationPoint.x},{annotationPoint.y})">
					<circle r="5" stroke="white" stroke-width="2" />
					<text x="10" y="-10" stroke="white" stroke-width="3" paint-order="stroke fill">
						{annotation.text}
					</text>
				</g>
			{/if}
		</svg>
	</div>

	<!-- legend -->
	<div class="legend">
		{#each species as sp (sp)}
			<span class="key" class:dim={highlight && highlight !== sp}>
				<span class="swatch" style="background:{colors[sp]}"></span>
				{sp}
			</span>
		{/each}
	</div>
</div>

<style>
	.background {
		display: flex;
		flex-direction: column;
		width: 100%;
		height: 100vh;
		box-sizing: border-box;
		font-family: system-ui, sans-serif;
		padding: 2rem;
	}

	.heading h2 {
		margin: 0;
		font-size: 1.1rem;
		color: #1d3557;
	}

	.heading p {
		margin: 0.15rem 0 0;
		font-size: 0.85rem;
		color: #888;
	}

	.chart {
		flex: 1;
		width: 100%;
		min-height: 0;
	}

	svg {
		width: 100%;
		height: 100%;
		display: block;
	}

	path {
		transition:
			d 0.5s ease,
			opacity 0.4s ease,
			stroke-width 0.4s ease;
	}

	.tick line {
		stroke: #e2e2e2;
	}

	.tick text {
		font-size: 11px;
		fill: #666;
	}

	svg text {
		font-size: 11px;
		fill: #666;
	}

	.annotation circle {
		fill: #1d3557;
	}

	.annotation text {
		font-size: 13px;
		font-weight: 600;
		fill: #1d3557;
	}

	.legend {
		display: flex;
		flex-wrap: wrap;
		gap: 1rem;
		font-size: 0.8rem;
		color: #444;
	}

	.key {
		display: flex;
		align-items: center;
		gap: 0.35rem;
		transition: opacity 0.3s ease;
	}

	.key.dim {
		opacity: 0.3;
	}

	.swatch {
		width: 12px;
		height: 12px;
		border-radius: 2px;
		display: inline-block;
	}
</style>
