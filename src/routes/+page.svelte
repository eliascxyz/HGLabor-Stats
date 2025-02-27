<script>
	import { onMount } from 'svelte';

	let stats = null;

	onMount(async () => {
		const response = await fetch(
			'https://api.hglabor.de/stats/ffa/26a4fcde-de39-4ff0-8ea1-786582b7d8ee'
		);
		stats = await response.json();
	});
</script>

{#if stats}
	<pre>{JSON.stringify(stats, null, 2)}</pre>
{:else}
	<p>Lade Stats...</p>
{/if}
