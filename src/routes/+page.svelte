<script>
	let leaderboardData = null;
	let error = '';
	let selectedSort;

	async function fetchLeaderboard() {
		error = '';
		leaderboardData = null;

		try {
			const response = await fetch(
				`https://api.hglabor.de/stats/FFA/top?sort=${selectedSort}&page=1`
			);
			if (!response.ok) throw new Error('Leaderboard could not be loaded.');
			leaderboardData = await response.json();
		} catch (err) {
			error = err.message;
		}
	}
	fetchLeaderboard();
</script>

<!-- Auswahl der Sortierung -->
<div class="mt-4 flex justify-center space-x-4">
	<select
		bind:value={selectedSort}
		on:change={fetchLeaderboard}
		class="rounded-lg border border-gray-300 bg-white px-3 py-2 text-gray-900 shadow-sm focus:ring-2 focus:ring-cyan-400 focus:outline-none"
	>
		<option value="kills">Kills</option>
		<option value="deaths">Deaths</option>
		<option value="xp">XP</option>
		<option value="bounty">Bounty</option>
		<option value="currentKillStreak">Killstreak</option>
		<option value="highestKillStreak">Highest KS</option>
	</select>
</div>

<!-- Fehleranzeige -->
{#if error}
	<p class="mt-2 text-center text-sm text-red-500">{error}</p>
{/if}

<!-- JSON Leaderboard Ausgabe -->
{#if leaderboardData}
	<pre
		class="mt-4 overflow-x-auto rounded-lg bg-gray-100 p-4 text-sm text-gray-900">{JSON.stringify(
			leaderboardData,
			null,
			2
		)}</pre>
{/if}
