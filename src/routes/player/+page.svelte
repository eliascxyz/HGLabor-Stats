<script>
	import { onMount } from 'svelte';

	let searchQuery = '';
	let stats = null;
	let heroStats = null;
	let error = '';

	async function getUUID(username) {
		try {
			const response = await fetch(`https://playerdb.co/api/player/minecraft/${username}`);
			if (!response.ok) throw new Error('Spieler nicht gefunden.');
			const data = await response.json();
			return data.data.player.id;
		} catch (err) {
			error = 'UUID konnte nicht gefunden werden.';
			return null;
		}
	}

	async function fetchStats() {
		if (!searchQuery) return;

		error = '';
		stats = null;
		heroStats = null;

		let uuid = searchQuery;
		if (!uuid.includes('-')) {
			uuid = await getUUID(searchQuery);
			if (!uuid) return;
		}

		try {
			const response = await fetch(`https://api.hglabor.de/stats/ffa/${uuid}`);
			if (!response.ok) throw new Error('Spieler nicht gefunden.');
			const data = await response.json();

			stats = {
				xp: data.xp,
				bounty: data.bounty,
				kills: data.kills,
				deaths: data.deaths,
				killStreak: data.currentKillStreak,
				highestKillStreak: data.highestKillStreak,
				uuid,
				playerName: searchQuery
			};

			heroStats = data.heroes;
		} catch (err) {
			error = err.message;
		}
	}
</script>

<!-- Überschrift -->
<h1 class="mt-8 text-center text-4xl font-bold text-gray-900">Hero FFA Stats</h1>

<!-- Suchleiste -->
<div class="mt-6 flex items-center justify-center space-x-4">
	<input
		type="text"
		bind:value={searchQuery}
		placeholder="Search for player"
		class="rounded-lg border-2 border-gray-300 bg-white p-2 text-gray-900 focus:ring-2 focus:ring-cyan-400 focus:outline-none"
	/>
	<button
		on:click={fetchStats}
		class="rounded-lg bg-cyan-400 px-4 py-2 font-bold text-white shadow-md transition hover:bg-cyan-500"
	>
		Search
	</button>
</div>

{#if error}
	<p class="mt-4 text-center text-red-500">{error}</p>
{/if}

<!-- Spieler-Stats -->
{#if stats}
	<div
		class="mx-auto mt-6 w-96 rounded-lg border border-gray-200 bg-white p-6 text-center shadow-lg"
	>
		<h2 class="text-xl font-semibold text-gray-900">{stats.playerName}</h2>
		<p class="text-gray-700">XP: {stats.xp}</p>
		<p class="text-gray-700">Bounty: {stats.bounty}</p>
		<p class="text-gray-700">Kills: {stats.kills}</p>
		<p class="text-gray-700">Deaths: {stats.deaths}</p>
		<p class="text-gray-700">
			K/D: {stats.deaths > 0 ? (stats.kills / stats.deaths).toFixed(2) : stats.kills}
		</p>
		<p class="text-gray-700">Killstreak: {stats.killStreak} (Max: {stats.highestKillStreak})</p>
	</div>
{/if}

<!-- Hero-Stats -->
{#if heroStats}
	<h2 class="mt-10 text-center text-2xl text-gray-900">Hero Stats</h2>
	<div class="mt-4 flex flex-wrap justify-center gap-4">
		{#each Object.entries(heroStats) as [hero, abilities]}
			<div class="w-80 rounded-lg border border-gray-200 bg-white p-4 shadow-md">
				<h3 class="text-lg font-semibold text-gray-900">{hero.replaceAll('_', ' ')}</h3>
				<ul class="mt-2 text-left text-gray-700">
					{#each Object.entries(abilities) as [ability, upgrades]}
						<li class="mt-2 font-medium text-cyan-600">{ability.replaceAll('_', ' ')}</li>
						<ul class="ml-4 text-gray-600">
							{#each Object.entries(upgrades) as [upgrade, details]}
								<li class="pl-2">{upgrade.replaceAll('_', ' ')}: {details.experiencePoints} XP</li>
							{/each}
						</ul>
					{/each}
				</ul>
			</div>
		{/each}
	</div>
{/if}
