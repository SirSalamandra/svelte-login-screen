<script lang="ts">
	import { onDestroy, onMount } from 'svelte';

	interface Props {
		onInputFocusChanged: (input: 'username' | 'password') => void;
		username: string;
		password: string;
		disableTyping: boolean;
	}

	let { onInputFocusChanged, username, password, disableTyping = false }: Props = $props();

	const handleKeyDown = (event: KeyboardEvent) => {
		event.preventDefault();
    return;
	};

  let eventListenerAdded = false;

	onMount(() => {
		document.getElementById('username')?.focus();

		if (disableTyping) {
			window.addEventListener('keydown', handleKeyDown);
      eventListenerAdded = true;
		}
	});

	onDestroy(() => {

    if (eventListenerAdded) {
      window.removeEventListener('keydown', handleKeyDown);
    }
	});
</script>

<div class="z-50 flex w-full max-w-sm flex-col space-y-4 rounded-lg bg-gray-800 p-8 shadow-lg">
	<h1 class="text-center text-2xl font-bold text-white">Login</h1>

	<input
		type="text"
		placeholder="Username"
		id="username"
		class="rounded-md border-2 border-gray-600 bg-gray-700 p-3 text-white placeholder-gray-400 transition focus:border-blue-500 focus:outline-none"
		onfocus={() => onInputFocusChanged('username')}
		bind:value={username}
	/>

	<input
		type="password"
		placeholder="Password"
		class="rounded-md border-2 border-gray-600 bg-gray-700 p-3 text-white placeholder-gray-400 transition focus:border-blue-500 focus:outline-none"
		onfocus={() => onInputFocusChanged('password')}
		bind:value={password}
	/>

	<button
		class="focus:ring-opacity-50 rounded-md bg-blue-600 py-3 font-bold text-white transition hover:bg-blue-700 focus:ring-2 focus:ring-blue-500 focus:outline-none"
	>
		Login
	</button>

	<div class="text-center text-white">Press CapsLock to toggle capital letters</div>
</div>
