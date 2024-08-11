<script lang="ts">
	import ShotSpinner from './ShotSpinner.svelte';

	import Button from '$lib/components/ui/button/button.svelte';
	import * as Card from '$lib/components/ui/card';

	import { Dice1, Dice2, Minus, Play, Plus, RefreshCcw, Trash, UserPlus } from 'lucide-svelte';

	import * as Table from '$lib/components/ui/table';

	import Input from '$lib/components/ui/input/input.svelte';

	type Player = {
		id: number;
		name: string;
	};

	type PlayerScore = {
		playerId: number;
		value: number;
	};

	type Round = {
		index: number;
		playerScore: PlayerScore[];
	};

	let players: Player[] = [];
	let rounds: Round[] = [];

	let currentRound = 0;
	let playerCount = 0;
	let showShot = false;

	function addPlayer(name: string | undefined): undefined {
		if (!name) return;

		let newPlayer: Player = {
			id: playerCount,
			name: name
		};
		players = [...players, newPlayer];

		playerCount++;
		playerName = undefined;
	}

	function removePlayer(id: number) {
		players = players.filter((player) => player.id !== id);
	}
	function startRound() {
		let newRound: Round = {
			index: currentRound,
			playerScore: players.map((player) => {
				return { playerId: player.id, value: 0 };
			})
		};

		currentRound++;
		rounds = [...rounds, newRound];
	}

	function resetGame() {
		players = [];
		rounds = [];
		playerCount = 0;
		currentRound = 0;
	}

	function score(id: number) {
		const playerScore = rounds[currentRound - 1].playerScore.find((score) => score.playerId === id);
		if (playerScore) {
			playerScore.value++;
			if (playerScore.value >= 5) {
				startRound();
				animateShot();
			}
			rounds = [...rounds];
		}
	}

	function removeScore(id: number) {
		const playerScore = rounds[currentRound - 1].playerScore.find((score) => score.playerId === id);
		if (playerScore && playerScore.value > 0) {
			playerScore.value--;
			rounds = [...rounds];
		}
	}

	function animateShot() {
		showShot = true;
		setTimeout(() => (showShot = false), 2000);
	}

	function scoreFormatter(score: number): string {
		return score < 5 ? 'l'.repeat(score) : 'llll';
	}

	let playerName: string | undefined;
</script>

<nav class="sticky top-0 z-10 flex items-center justify-between border bg-white p-2">
	<h1 class="font- flex uppercase"><Dice2 /><Dice1 /> <span class="pl-2">Mäxchen</span></h1>

	<div class="flex items-center gap-2">
		{#if rounds.length > 0}
			<Button variant="outline" size="icon" on:click={() => resetGame()}>
				<RefreshCcw class="h-4 w-4" />
			</Button>
		{/if}
	</div>
</nav>
<main class="p-4">
	{#if rounds.length === 0}
		<Card.Root>
			<Card.Header>
				<Card.Title>Add Players</Card.Title>
				<Card.Description>Add a new players to the Game</Card.Description>
			</Card.Header>
			<Card.Content>
				<form on:submit={() => addPlayer(playerName)}>
					<div class="grid gap-2">
						{#each players as player}
							<div class="flex justify-stretch gap-2">
								<div class="w-full rounded-md border px-4 py-3 font-mono text-sm">
									{player.name}
								</div>
								<Button
									variant="ghost"
									class="hover:bg-red-50"
									size="icon"
									on:click={() => removePlayer(player.id)}
								>
									<Trash class="h-4 w-4 text-red-500" />
								</Button>
							</div>
						{/each}
						<div class="mt-2 flex justify-stretch gap-2">
							<Input placeholder="Name" id="name" bind:value={playerName} />
							<Button variant="secondary" type="submit">
								<UserPlus class="mr-2 h-4 w-4" /> Add
							</Button>
						</div>
					</div>
				</form>
			</Card.Content>
		</Card.Root>

		{#if players.length > 0}
			<Button class="mt-4 w-full" on:click={() => startRound()}>
				<Play class="mr-2 h-4 w-4" /> Start Game
			</Button>
		{/if}
		<div class="flex w-full items-center justify-center gap-2 p-4"></div>
	{/if}

	{#if rounds.length > 0}
		<Table.Root>
			<Table.Header>
				<Table.Row>
					<Table.Head class="w-[20px]">#</Table.Head>
					{#each players as player}
						<Table.Head>{player.name}</Table.Head>
					{/each}
				</Table.Row>
			</Table.Header>
			<Table.Body>
				{#each rounds as round}
					<Table.Row>
						<Table.Cell>{round.index}</Table.Cell>
						{#each round.playerScore as playerScore}
							<Table.Cell>
								<span
									class="tracking-widest"
									class:text-red-800={playerScore.value === 4}
									class:text-red-500={playerScore.value === 5}
									class:line-through={playerScore.value === 5}
								>
									{scoreFormatter(playerScore.value)}</span
								>
							</Table.Cell>
						{/each}
					</Table.Row>
				{/each}
				{#if rounds.length > 0}
					<Table.Row>
						<Table.Cell />
						{#each players as player}
							<Table.Cell>
								<Button variant="outline" size="icon" on:click={() => score(player.id)}>
									<Plus class="h-4 w-4" />
								</Button>

								<Button variant="outline" size="icon" on:click={() => removeScore(player.id)}>
									<Minus class="h-4 w-4" />
								</Button>
							</Table.Cell>
						{/each}
					</Table.Row>
				{/if}
			</Table.Body>
		</Table.Root>
	{/if}

	<ShotSpinner show={showShot} />
</main>
