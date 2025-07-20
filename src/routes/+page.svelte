<script lang="ts">
	import '../app.css'; 
	import GameDisplay from '$lib/GameDisplay.svelte';
</script>

<style>
	html, body {
		margin: 0;
		padding: 0;
		height: 100%;
		width: 100%;
		overflow: hidden;
	}

	.fullscreen-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		margin: 0;
		padding: 0;
		background: black;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>

<div class="fullscreen-container">
	<GameDisplay />
</div>
