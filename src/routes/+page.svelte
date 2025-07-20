<script lang="ts">
	import '../app.css'; 
	import GameDisplay from '$lib/GameDisplay.svelte';
</script>

<svelte:head>
	<style>
		html, body, #svelte, #sveltekit-body {
			width: 100%;
			height: 100%;
			margin: 0;
			padding: 0;
			overflow: hidden;
		}
	</style>
</svelte:head>

<style>
	.fullscreen-container {
		position: fixed;
		inset: 0;
		width: 100vw;
		height: 100vh;
		background: black;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>

<div class="fullscreen-container">
	<GameDisplay />
</div>
