<script lang="ts">
	import { onDestroy, onMount } from 'svelte';
	import ClickLetter from '../components/click-letter.svelte';

	let letterDropInterval = 500; // milliseconds
	let letterFallDuration = 5000; // milliseconds

	let revealPassword = false;
	let useCapitalLetter = false;

	let username = '';
	let password = '';
	let lastInputSelected: 'username' | 'password' = 'username';

	function pickRandomAlphanumeric() {
		const chars = 'abcdefghijklmnopqrstuvwxyz0123456789';

		const pickedChar = chars.charAt(Math.floor(Math.random() * chars.length));

		return useCapitalLetter ? pickedChar.toUpperCase() : pickedChar;
	}

	let letters: { id: number; char: string; left: number }[] = [];
	let nextId = 0;

	let letterElements = new Map<number, HTMLDivElement>();

	function capitalLetterToggle(event: KeyboardEvent) {
		if (event.key !== 'CapsLock') {
			event.preventDefault();
			return;
		}

		event.getModifierState('CapsLock') ? (useCapitalLetter = true) : (useCapitalLetter = false);
	}

	const dropLetterIntervalId = setInterval(() => {
		const newLetter = {
			id: nextId++,
			char: pickRandomAlphanumeric(),
			left: Math.max(5, Math.random() * 95),
			visible: lastInputSelected === 'password' ? false : true
		};

		letters = [...letters, newLetter];

		setTimeout(() => {
			letters = letters.filter((l) => l.id !== newLetter.id);
			letterElements.delete(newLetter.id);
		}, letterFallDuration);
	}, letterDropInterval);

	const checkCollisionIntervalId = setInterval(() => {
		checkCollision();
	}, 50);

	let isEventListenerAdded = false;

	onMount(() => {
		window.addEventListener('keydown', capitalLetterToggle);
		isEventListenerAdded = true;

		document.getElementById('username')?.focus();

		deleteLeft = Math.random() * (window.innerWidth - 100);
		deleteTop = Math.random() * (window.innerHeight - 100);
	});

	onDestroy(() => {
		clearInterval(dropLetterIntervalId);
		clearInterval(checkCollisionIntervalId);

		if (isEventListenerAdded) {
			window.removeEventListener('keydown', capitalLetterToggle);
		}
	});

	function checkCollision() {
		const barRect = document.querySelector('.bar')?.getBoundingClientRect();
		if (!barRect) return;

		letterElements.forEach((el, id) => {
			const letterRect = el.getBoundingClientRect();

			if (
				letterRect.bottom >= barRect.top &&
				letterRect.top <= barRect.bottom &&
				letterRect.right >= barRect.left &&
				letterRect.left <= barRect.right
			) {
				console.log(`Caught letter: ${letters.find((l) => l.id === id)?.char}`);

				if (lastInputSelected === 'username') {
					username += letters.find((l) => l.id === id)?.char || '';
				} else {
					password += letters.find((l) => l.id === id)?.char || '';
				}

				letters = letters.filter((l) => l.id !== id);
				letterElements.delete(id);
			}
		});
	}

	let barLeft = 0;
	let barWidth = 0;

	function handleMouseMove(event: PointerEvent) {
		let newLeft = event.clientX - barWidth / 2;
		barLeft = Math.max(0, Math.min(newLeft, window.innerWidth - barWidth));
	}

	function registerLetterElement(el: HTMLDivElement, id: number) {
		letterElements.set(id, el);
	}

	let deleteLeft = 0;
	let deleteTop = 0;

	function handleDelete() {
		if (lastInputSelected === 'username') {
			username = username.slice(0, -1);
		} else {
			password = password.slice(0, -1);
		}

		deleteLeft = Math.random() * (window.innerWidth - 100);
		deleteTop = Math.random() * (window.innerHeight - 100);
	}

	function handleInputFocused(input: 'username' | 'password') {
		lastInputSelected = input;
		// changeLettersVisibility(input === 'password' ? false : true);
	}

	// function changeLettersVisibility(showLetters: boolean) {
	// 	revealPassword = showLetters;
	// 	letters = letters.map((l) => ({ ...l, visible: showLetters }));
	// }
</script>

<div onpointermovecapture={handleMouseMove} style="--fall-duration: {letterFallDuration}ms;">
	<div class="flex h-screen w-screen items-center justify-center bg-gray-900">
		<div class="flex w-full max-w-sm flex-col space-y-4 rounded-lg bg-gray-800 p-8 shadow-lg">
			<h1 class="text-center text-2xl font-bold text-white">Login</h1>

			<input
				type="text"
				placeholder="Username"
				id="username"
				class="rounded-md border-2 border-gray-600 bg-gray-700 p-3 text-white placeholder-gray-400 transition focus:border-blue-500 focus:outline-none"
				onfocuscapture={handleInputFocused.bind(null, 'username')}
				bind:value={username}
			/>

			<input
				type="password"
				placeholder="Password"
				class="rounded-md border-2 border-gray-600 bg-gray-700 p-3 text-white placeholder-gray-400 transition focus:border-blue-500 focus:outline-none"
				onfocuscapture={handleInputFocused.bind(null, 'password')}
				bind:value={password}
			/>

			<!-- <button onclick={changeLettersVisibility.bind(null, !revealPassword)}>
				{#if revealPassword}
					<span class="text-gray-400 hover:text-gray-200">Hide</span>
				{:else}
					<span class="text-gray-400 hover:text-gray-200">Show</span>
				{/if}
			</button> -->

			<button
				class="focus:ring-opacity-50 rounded-md bg-blue-600 py-3 font-bold text-white transition hover:bg-blue-700 focus:ring-2 focus:ring-blue-500 focus:outline-none"
			>
				Login
			</button>

			<div class="text-center text-white">Press CapsLock to toggle capital letters</div>
		</div>

	</div>

	<!-- {#each letters as letter (letter.id)}
		<div style:left="{letter.left}vw" class="falling-object" use:registerLetterElement={letter.id}>
				{letter.char}
		</div>
	{/each} -->

	<div class="bar" style:left="{barLeft}px" bind:clientWidth={barWidth}></div>

	{#if username || password}
		<button
			class="delete-button"
			style:left="{deleteLeft}px"
			style:top="{deleteTop}px"
      use:registerLetterElement={letter.id}
			onclick={handleDelete}>delete</button
		>
	{/if}
</div>

<style>
	.delete-button {
		position: absolute;
		top: 20px;
		right: 20px;
		padding: 10px 20px;
		background-color: #ef4444;
		color: white;
		border: none;
		border-radius: 5px;
		cursor: pointer;
		font-weight: bold;
		transition: background-color 0.3s ease;
		width: min-content;
	}

	.bar {
		position: absolute;
		bottom: 20px;
		width: 100px;
		height: 20px;
		background-color: #0ea5e9;
		border-radius: 5px;
	}

	.falling-object {
		z-index: 10;
		position: absolute;
		color: white;
		font-size: 1.5rem;
		top: -50px;
		animation: fall var(--fall-duration) linear;
		user-select: none;
	}

	@keyframes fall {
		from {
			/* The 'from' state is already set in .falling-object */
			/* No change needed here, but you could define it explicitly */
		}
		to {
			/* Moves the object to just below the bottom of the screen */
			transform: translateY(100vh);
		}
	}
</style>
