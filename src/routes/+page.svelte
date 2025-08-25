<script lang="ts">
	import { onDestroy, onMount } from 'svelte';
	import ClickLetter from '../components/click-letter.svelte';
	import DeleteButton from '../components/delete-button.svelte';
	import Login from '../components/login.svelte';

	let useCapitalLetter = $state(false);
	let selectedInput = $state<'username' | 'password'>('username');
	let username = $state('');
	let password = $state('');

	function handleLetterClick(letter: string) {
		if (selectedInput === 'username') {
			username += letter;
		} else {
			password += letter;
		}
	}

	function handleDeleteLetter() {
		if (selectedInput === 'username') {
			username = username.slice(0, -1);
		} else {
			password = password.slice(0, -1);
		}
	}

	function handleInputFocusChanged(input: 'username' | 'password') {
		selectedInput = input;
	}

	function capitalLetterToggle(event: KeyboardEvent) {
		if (event.key !== 'CapsLock') {
			event.preventDefault();
			return;
		}

		event.getModifierState('CapsLock') ? (useCapitalLetter = true) : (useCapitalLetter = false);
	}

	let isEventListenerAdded = false;

	onMount(() => {
		window.addEventListener('keydown', capitalLetterToggle);
		isEventListenerAdded = true;
	});

	onDestroy(() => {
		if (isEventListenerAdded) {
			window.removeEventListener('keydown', capitalLetterToggle);
		}
	});
</script>

<div class="flex h-screen w-screen items-center justify-center bg-gray-900">
	<Login disableTyping {username} {password} onInputFocusChanged={handleInputFocusChanged} />

	{#if username || password}
		<DeleteButton onDelete={handleDeleteLetter} />
	{/if}

	<ClickLetter onLetterClick={handleLetterClick} capitalLetter={useCapitalLetter} />
</div>
