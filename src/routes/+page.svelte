<script lang="ts">
	import ShotSpinner from './ShotSpinner.svelte';

	import Button from '$lib/components/ui/button/button.svelte';
	import { Label } from '$lib/components/ui/label/index.js';

	import { Dice1, Dice2, Minus, Play, Plus, RefreshCcw } from 'lucide-svelte';

	import * as Drawer from '$lib/components/ui/drawer';
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
	let isAddPlayerOpen = false;
</script>

<nav class="flex sticky top-0 items-center justify-between p-2 z-10 bg-white border">
	<h1 class="font- uppercase flex"><Dice2 /><Dice1 /> <span class="pl-2">Mäxchen</span></h1>

	<div class="flex items-center gap-2">
		{#if rounds.length > 0}
			<Button variant="outline" size="icon" on:click={() => resetGame()}>
				<RefreshCcw class="h-4 w-4" />
			</Button>
		{/if}
	</div>
</nav>
<main class="relative">
	{#if rounds.length === 0}
		<div class="p-4 w-full flex gap-2 items-center justify-center">
			<Drawer.Root bind:open={isAddPlayerOpen}>
				<Drawer.Trigger asChild let:builder>
					<Button variant="outline" builders={[builder]}>Add Players</Button>
				</Drawer.Trigger>
				<Drawer.Content>
					<Drawer.Header class="text-left">
						<Drawer.Title>Add Player</Drawer.Title>
						<Drawer.Description>Add a new player to the Game</Drawer.Description>
					</Drawer.Header>
					<form class="grid items-start gap-4 px-4" on:submit={() => addPlayer(playerName)}>
						<div class="grid gap-2">
							<Label for="name">Name</Label>
							<Input id="name" bind:value={playerName} />
						</div>

						<Button type="submit">Add</Button>
					</form>
					<Drawer.Footer class="pt-2">
						<Drawer.Close>Cancel</Drawer.Close>
					</Drawer.Footer>
				</Drawer.Content>
			</Drawer.Root>

			{#if players.length > 0}
				<Button variant="outline" on:click={() => startRound()}>
					<Play class="h-4 w-4 mr-2" /> Start Game
				</Button>
			{/if}
		</div>
	{/if}

	{#if players.length > 0}
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
