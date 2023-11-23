<script>
	import GameCard, { randomCard } from '$lib/GameCard.svelte';
	import EmptySpot from '$lib/EmptySpot.svelte';
	import Grid from '$lib/Grid.svelte';
	import Stack from '$lib/Stack.svelte';

	/** @type {Array<Card | undefined>}*/
	let game = $state([
		undefined,
		undefined,
		undefined,
		undefined,
		undefined,
		undefined,
		undefined,
		undefined,
		undefined
	]);
	/** @type {Array<number | undefined>} */
	let cardOwners = $state([
		undefined,
		undefined,
		undefined,
		undefined,
		undefined,
		undefined,
		undefined,
		undefined,
		undefined
	]);

	/** @type {boolean} */
	let isPlayer2Turn = $state(Math.random() < 0.5);

	/** @type {Card | undefined} */
	let selectedCard = $state(undefined);

	/** @type {[Array<Card | undefined>, Array<Card | undefined>]}*/
	let decks = $state([
		Array.from(new Array(5)).map(randomCard),
		Array.from(new Array(5)).map(randomCard)
	]);

	/** @param {number} index */
	function applyCard(index) {
		// Not super clean, but it works
		const card = game[index];
		if (card) {
			const [left, right, top, bottom, _flags] = card;
			if (index <= 5) {
				const competingIndex = index + 3;
				const competing = game[competingIndex];
				if (competing) {
					const [, , topC] = competing;
					if (bottom > topC) {
						switchOwner(competingIndex);
					}
				}
			}
			if (index > 3) {
				const competingIndex = index - 3;
				const competing = game[competingIndex];
				if (competing) {
					const [, , , bottomC] = competing;
					if (top > bottomC) {
						switchOwner(competingIndex);
					}
				}
			}
			if (index !== 0 && index !== 3 && index !== 6) {
				const competingIndex = index - 1;
				const competing = game[competingIndex];
				if (competing) {
					const [, rightC] = competing;
					if (left > rightC) {
						switchOwner(competingIndex);
					}
				}
			}
			if (index !== 2 && index !== 5 && index !== 8) {
				const competingIndex = index + 1;
				const competing = game[competingIndex];
				if (competing) {
					const [leftC] = competing;
					if (right > leftC) {
						switchOwner(competingIndex);
					}
				}
			}
		}
	}

	/** @param {number} index */
	function switchOwner(index) {
		cardOwners = cardOwners.map((owner, i) => (i === index ? (owner === 1 ? 2 : 1) : owner));
	}

	/** @param {number} index */
	function select(index) {
		if (selectedCard) {
			if (isPlayer2Turn) {
				decks = [decks[0], decks[1].map((c) => (c === selectedCard ? undefined : c))];
			} else {
				decks = [decks[0].map((c) => (c === selectedCard ? undefined : c)), decks[1]];
			}
			game = game.map((card, i) => (i === index ? selectedCard : card));
			cardOwners = cardOwners.map((o, i) => (i === index ? (isPlayer2Turn ? 2 : 1) : o));
			applyCard(index);
			isPlayer2Turn = !isPlayer2Turn;
			selectedCard = undefined;
		}
	}

	/** @param {Card} card */
	function selectP1(card) {
		if (!isPlayer2Turn) {
			selectedCard = card;
		}
	}

	/** @param {Card} card */
	function selectP2(card) {
		if (isPlayer2Turn) {
			selectedCard = card;
		}
	}

	function reset() {
		game = Array.from(new Array(9)).map(() => undefined);
		cardOwners = Array.from(new Array(9)).map(() => undefined);
		selectedCard = undefined;
		isPlayer2Turn = Math.random() < 0.5;
		decks = [Array.from(new Array(5)).map(randomCard), Array.from(new Array(5)).map(randomCard)];
	}

	let isFinished = $derived(game.every((c) => c !== undefined));
</script>

<div class="h-screen w-screen">
	<div class="flex justify-center">Turn: {isPlayer2Turn ? 'P2' : 'P1'}</div>

	<div class="absolute left-0 top-0">
		<Stack>
			{#each decks[0] as card}
				{#if card}
					<GameCard
						{card}
						selectable={!isPlayer2Turn}
						onclick={() => card && selectP1(card)}
						selected={card === selectedCard}
						color="blue"
					/>
				{/if}
			{/each}
		</Stack>
	</div>
	<div class="flex items-center justify-center">
		<div class="w-96 h-96">
			<Grid>
				{#each game as card, index}
					{#if card}
						<GameCard {card} color={cardOwners[index] === 1 ? 'blue' : 'red'} />
					{:else}
						<EmptySpot onclick={() => select(index)} />
					{/if}
				{/each}
			</Grid>
		</div>
	</div>
	<div class="absolute right-0 top-0">
		<Stack>
			{#each decks[1] as card}
				{#if card}
					<GameCard
						{card}
						selectable={isPlayer2Turn}
						onclick={() => card && selectP2(card)}
						selected={card === selectedCard}
						color="red"
					/>
				{/if}
			{/each}
		</Stack>
	</div>

	<div class="flex flex-col items-center gap-2">
		{#if isFinished}
			<div>YA DONE</div>
			<button onclick={reset}>Reset?</button>
		{/if}
	</div>
</div>
