<script>
	import { onMount } from 'svelte';

	let currentPage = 1;
	let totalPages = 5;
	let allPlayers = [];
	let playerNames = {};
	let error = '';
	let loading = true;
	let selectedSort = 'kills';

	onMount(getAllPlayers);

	async function getAllPlayers() {
		try {
			loading = true;
			const players = await getPage();
			allPlayers = players;
			totalPages = Math.ceil(players.length / 10);

			await fetchPlayerNames(players); // Spielernamen abrufen

			showPage(currentPage);
		} catch (err) {
			error = 'Error while loading Leaderboard.';
			console.error(err);
		} finally {
			loading = false;
		}
	}

	async function fetchPlayerNames(players) {
		const nameRequests = players.map(async (player) => {
			const name = await getPlayer(player.playerId);
			playerNames = { ...playerNames, [player.playerId]: name };
		});

		await Promise.all(nameRequests); // Alle Anfragen parallel abwarten
	}

	async function getPlayer(uuid) {
		try {
			const response = await fetch(`https://playerdb.co/api/player/minecraft/${uuid}`);
			if (!response.ok) throw new Error('Player not found.');
			const data = await response.json();
			return data.data.player.username;
		} catch (err) {
			console.error('Fehler beim Abrufen des Spielernamens:', err);
			return 'Unbekannt';
		}
	}

	function showPage(page) {
		const start = (page - 1) * 10;
		const end = start + 10;
		return allPlayers.slice(start, end);
	}

	async function getPage() {
		const response = await fetch(
			`https://api.hglabor.de/stats/ffa/top?sort=${selectedSort}&page=1`
		);
		if (!response.ok) throw new Error(`Error: ${response.status} ${response.statusText}`);
		return await response.json();
	}

	function changePage(direction) {
		const newPage = currentPage + direction;
		if (newPage >= 1 && newPage <= totalPages) {
			currentPage = newPage;
		}
	}

	function changeSort(event) {
		getAllPlayers();
		currentPage = 1; // Zurück auf Seite 1 nach Sortierung
	}
</script>

<div class="flex flex-col items-center">
	{#if error}
		<p class="mt-4 text-center text-red-500">{error}</p>
	{/if}

	{#if loading}
		<p class="mt-4 text-gray-700">Loading Leaderboard...</p>
	{/if}

	<div class="mt-4">
		<label for="sort" class="mr-2 font-semibold text-gray-900">Sort by:</label>
		<select
			id="sort"
			bind:value={selectedSort}
			on:change={changeSort}
			class="rounded-lg border-2 border-gray-300 bg-white p-2 text-gray-900 focus:ring-2 focus:ring-cyan-400 focus:outline-none"
		>
			<option value="kills">Kills</option>
			<option value="deaths">Deaths</option>
			<option value="bounty">Bounty</option>
		</select>
	</div>

	{#if !loading && allPlayers.length > 0}
		<div class="mt-6">
			<table class="w-full rounded-lg border border-gray-300 bg-white shadow-md">
				<thead class="bg-cyan-400 text-white">
					<tr>
						<th class="px-4 py-2 text-left">#</th>
						<th class="px-4 py-2 text-left">Avatar</th>
						<th class="px-4 py-2 text-left">Player</th>
						<th class="px-4 py-2 text-left">Kills</th>
						<th class="px-4 py-2 text-left">Deaths</th>
						<th class="px-4 py-2 text-left">Bounty</th>
					</tr>
				</thead>
				<tbody>
					{#each showPage(currentPage) as player, index}
						<tr class="cursor-pointer border-b transition hover:bg-gray-100">
							<td class="px-4 py-2">{(currentPage - 1) * 10 + index + 1}</td>
							<td class="px-4 py-2">
								<img
									class="h-10 w-10 rounded-lg"
									src={`https://crafatar.com/avatars/${player.playerId}`}
									alt="Avatar"
								/>
							</td>
							<td class="px-4 py-2">{playerNames[player.playerId] || 'Lade...'}</td>
							<td class="px-4 py-2">{player.kills}</td>
							<td class="px-4 py-2">{player.deaths}</td>
							<td class="px-4 py-2">{player.bounty}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>

		<div class="mt-4 flex items-center justify-center space-x-4">
			<button
				on:click={() => changePage(-1)}
				class="rounded-lg bg-gray-300 px-4 py-2 text-gray-700 shadow-md hover:bg-gray-400 disabled:opacity-50"
				disabled={currentPage === 1}>Back</button
			>
			<span class="font-semibold text-gray-900">Page {currentPage} of {totalPages}</span>
			<button
				on:click={() => changePage(1)}
				class="rounded-lg bg-gray-300 px-4 py-2 text-gray-700 shadow-md hover:bg-gray-400 disabled:opacity-50"
				disabled={currentPage === totalPages}>Next</button
			>
		</div>
	{/if}
</div>
