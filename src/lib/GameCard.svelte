<script context="module">
	/** @param {number} max
	 * @param {number} min
	 */
	function randomRange(min, max) {
		return Math.floor(Math.random() * (max - min) + min);
	}

	/** @return {Card} */
	export function randomCard() {
		return [randomRange(1, 5), randomRange(1, 5), randomRange(1, 5), randomRange(1, 5), 0];
	}
</script>

<script>
	/** @type {{ card: Card, onclick?: () => void, selectable?: boolean, selected?: boolean, color: 'blue' | 'red' | 'zinc' }} */
	const { card, onclick, selectable, selected, color } = $props();
	const [left, right, top, bottom, flags] = card;
</script>

<div
	class="relative border-2 aspect-square"
	class:bg-red-100={color === 'red'}
	class:bg-blue-100={color === 'blue'}
	class:hover:cursor-pointer={selectable}
	class:border-blue-500={selected}
	{onclick}
>
	<div class="number left-2 h-full flex items-center">{left}</div>
	<div class="number right-2 h-full flex items-center">{right}</div>
	<div class="number top-2 w-full text-center">{top}</div>
	<div class="number bottom-2 w-full text-center">{bottom}</div>
</div>

<style>
	.number {
		@apply absolute pointer-events-none select-none;
	}
</style>
