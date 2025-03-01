<script>
	import '../app.css';
	let { children } = $props();
</script>

<nav class="flex items-center justify-between bg-white p-4 text-black shadow-md">
	<a href="/" class="flex items-center space-x-4">
		<img src="favicon.png" alt="HGLabor.de" class="h-12 w-12" />
		<span class="text-lg">HGLabor</span>
	</a>
	<div class="flex space-x-8">
		<a href="/" class="transition hover:text-cyan-400">Leaderboard</a>
		<a href="/player" class="transition hover:text-cyan-400">Player</a>
	</div>
</nav>

{@render children()}
