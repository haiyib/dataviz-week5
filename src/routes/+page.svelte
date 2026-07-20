<script>
	import Scroller from '$lib/components/layout/Scroller.svelte';
	import VizBackground from '$lib/components/demo/week5/VizBackground.svelte';
	import Slide from '$lib/components/demo/week5/Slide.svelte';

	const slides = [
		{
			title: 'Total Intake',
			description: 'Total animals taken into the shelter each year, broken out by species.'
		},
		{
			title: 'Stray / At Large',
			description: 'Scroll to swap the metric: strays make up the bulk of intake.'
		},
		{
			title: 'Adoptions',
			description: 'Live outcomes — how many of each species found a new home.'
		},
		{
			title: 'Highlight Cats',
			description: 'Later steps can dim other species and emphasize one line.'
		},
		{
			title: 'Dogs in 2024',
			description: 'The final step adds an annotation tied to a specific data point.'
		}
	];

	let index = $state(0);
	let offset = $state(0);
	let progress = $state(0);
	let count = $state(0);
</script>

<div class="scroller-demo">
	<Scroller
		top={0}
		bottom={1}
		ref="viz-background"
		bind:index
		bind:offset
		bind:progress
		bind:count
	>
		{#snippet background()}
			<VizBackground {index} {count} />
		{/snippet}

		{#snippet foreground()}
			{#each slides as slide}
				<Slide title={slide.title} description={slide.description} />
			{/each}
		{/snippet}
	</Scroller>
</div>

<style>
	.scroller-demo {
		max-width: none;
		margin: 0 -1.25rem;
	}
</style>
