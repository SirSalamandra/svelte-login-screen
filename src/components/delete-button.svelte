<script lang="ts">
	import { onMount } from "svelte";

  interface Props {
    onDelete: () => void;
  }

  let { onDelete }: Props = $props();

  let deleteLeft = $state(0);
	let deleteTop = $state(0);

	function handleDelete() {
    onDelete();

		deleteLeft = Math.random() * (window.innerWidth - 100);
		deleteTop = Math.random() * (window.innerHeight - 100);
	}

  onMount(() => {
    deleteLeft = Math.random() * (window.innerWidth - 100);
		deleteTop = Math.random() * (window.innerHeight - 100);
  })
</script>

<button
  style:top={deleteTop + 'px'}
  style:left={deleteLeft + 'px'}
  class="delete-button"
  onclick={handleDelete}
>Delete</button>

<style>
	.delete-button {
    z-index: 50;
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
</style>