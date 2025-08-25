<script lang="ts">
	import { onDestroy } from 'svelte';

	interface Props {
		onLetterClick: (letter: string) => void;
    capitalLetter: boolean;
	}

	type Letter = {
		id: number;
		char: string;
		left: number;
	};

	let { onLetterClick, capitalLetter }: Props = $props();

	let letterDropInterval = 250;
	let letterFallDuration = 6000;

	function pickRandomAlphanumeric() {
		const chars = 'abcdefghijklmnopqrstuvwxyz0123456789_@';

		const pickedChar = chars.charAt(Math.floor(Math.random() * chars.length));
		return capitalLetter ? pickedChar.toUpperCase() : pickedChar;
	}

	function capitalLetterToggle(event: KeyboardEvent) {
		if (event.key !== 'CapsLock') {
			event.preventDefault();
			return;
		}

		event.getModifierState('CapsLock') ? (capitalLetter = true) : (capitalLetter = false);
	}

	let nextId = 0;
	let letters: Letter[] = $state([]);
	let letterElements = new Map<number, HTMLButtonElement>();

	const dropLetterIntervalId = setInterval(() => {
		const newLetter = {
			id: nextId++,
			char: pickRandomAlphanumeric(),
			left: Math.max(5, Math.random() * 95)
		};

		letters = [...letters, newLetter];

		setTimeout(() => {
			letters = letters.filter((l) => l.id !== newLetter.id);
			letterElements.delete(newLetter.id);
		}, letterFallDuration);
	}, letterDropInterval);

	let isEventListenerAdded = false;

	onDestroy(() => {
		clearInterval(dropLetterIntervalId);

		if (isEventListenerAdded) {
			window.removeEventListener('keydown', capitalLetterToggle);
		}
	});

	function addLetterElement(el: HTMLButtonElement, id: number) {
		letterElements.set(id, el);
	}

	function handleClick(letter: Letter) {
		letterElements.delete(letter.id);
		letters = letters.filter((l) => l.id !== letter.id);

		onLetterClick(letter.char);
	}
</script>

<div class="absolute h-full w-full overflow-hidden" style="--fall-duration: {letterFallDuration}ms">
	{#each letters as letter (letter.id)}
		<button
			style:left="{letter.left}vw"
			class="falling-object"
			use:addLetterElement={letter.id}
			onclick={handleClick.bind(null, letter)}>{letter.char}</button
		>
	{/each}
</div>

<style>
	.falling-object {
		position: absolute;
		top: 0;
		line-height: 35px;
		width: 35px;
		font-size: 1.5rem;
		font-weight: bold;
		animation: fall var(--fall-duration) linear;
		user-select: none;
		background: none;
		border: none;
		color: white;
		cursor: pointer;
		text-align: center;
	}

	.falling-object:hover {
		background-color: rgba(165, 217, 252, 0.5);
		border-radius: 5px;
	}

	@keyframes fall {
		from {
		}
		to {
			top: 100vh;
		}
	}
</style>
