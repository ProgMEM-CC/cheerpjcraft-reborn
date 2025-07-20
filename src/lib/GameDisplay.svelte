<script lang="ts">
	import { onMount } from 'svelte';
	import { tryPlausible, showElement, hideElement } from "./utilities";
	import spinnerWhite from '$lib/assets/loading-spinner-white.svg';
	import ghGIF from '$lib/assets/rate-us-on-gh.gif';

	const pathJarMinecraft = "/files/client_1.2.5.jar";
	const urlDownloadMinecraft = "https://piston-data.mojang.com/v1/objects/4a2fac7504182a97dcbcd7560c6392d7c8139928/client.jar";
	const pathJarLibs = `/app/lwjgl/lwjgl-2.9.3.jar:/app/lwjgl/lwjgl_util-2.9.3.jar:${pathJarMinecraft}`;

	let loading: HTMLDivElement;
	let display: HTMLDivElement;
	let intro: HTMLDivElement;
	let progressBar: HTMLProgressElement;
	let eulaAccepted = false;

	async function startCheerpJ() {
		await cheerpjInit({
			version: 8,
			javaProperties: ["java.library.path=/app/lwjgl/libraries/"],
			libraries: { "libGL.so.1": "/app/lwjgl/libraries/gl4es.wasm" },
			enableX11: true,
			preloadResources: {
				"/lt/8/jre/lib/rt.jar": [0, 131072, 1310720, 1572864, 4456448, 4849664, 5111808, 5505024, 7995392, 8126464, 9699328, 9830400, 9961472, 11534336, 11665408, 12189696, 12320768, 12582912, 13238272, 13369344, 15073280, 15335424, 15466496, 15597568, 15990784, 16121856, 16252928, 16384000, 16777216, 16908288, 17039360, 17563648, 17694720, 17825792, 17956864, 18087936, 18219008, 18612224, 18743296, 18874368, 19005440, 19136512, 19398656, 19791872, 20054016, 20709376, 20840448, 21757952, 21889024, 26869760],
				"/lt/etc/users": [0, 131072],
				"/lt/etc/localtime": [],
				"/lt/8/jre/lib/cheerpj-awt.jar": [0, 131072],
				"/lt/8/lib/ext/meta-index": [0, 131072],
				"/lt/8/lib/ext": [],
				"/lt/8/lib/ext/index.list": [],
				"/lt/8/lib/ext/localedata.jar": [],
				"/lt/8/jre/lib/jsse.jar": [0, 131072, 786432, 917504],
				"/lt/8/jre/lib/jce.jar": [0, 131072],
				"/lt/8/jre/lib/charsets.jar": [0, 131072, 1703936, 1835008],
				"/lt/8/jre/lib/resources.jar": [0, 131072, 917504, 1179648],
				"/lt/8/jre/lib/javaws.jar": [0, 131072, 1441792, 1703936],
				"/lt/8/lib/ext/sunjce_provider.jar": [],
				"/lt/8/lib/security/java.security": [0, 131072],
				"/lt/8/jre/lib/meta-index": [0, 131072],
				"/lt/8/jre/lib": [],
				"/lt/8/lib/accessibility.properties": [],
				"/lt/8/lib/fonts/LucidaSansRegular.ttf": [],
				"/lt/8/lib/currency.data": [0, 131072],
				"/lt/8/lib/currency.properties": [],
				"/lt/libraries/libGLESv2.so.1": [0, 262144],
				"/lt/libraries/libEGL.so.1": [0, 262144],
				"/lt/8/lib/fonts/badfonts.txt": [],
				"/lt/8/lib/fonts": [],
				"/lt/etc/hosts": [],
				"/lt/etc/resolv.conf": [0, 131072],
				"/lt/8/lib/fonts/fallback": [],
				"/lt/fc/fonts/fonts.conf": [0, 131072],
				"/lt/fc/ttf": [],
				"/lt/fc/cache/e21edda6a7db77f35ca341e0c3cb2a22-le32d8.cache-7": [0, 131072],
				"/lt/fc/ttf/LiberationSans-Regular.ttf": [0, 131072, 262144, 393216],
				"/lt/8/lib/jaxp.properties": [],
				"/lt/etc/timezone": [],
				"/lt/8/lib/tzdb.dat": [0, 131072]
			}
		});
		await cheerpjCreateDisplay(-1, -1, display);

		hideElement(loading);
		showElement(intro);
	}

	async function startGame() {
		hideElement(intro);
		showElement(progressBar);

		await downloadFileToCheerpJ();
		hideElement(progressBar);
		showElement(display);

		tryPlausible("Play");
		await cheerpjRunMain("net.minecraft.client.Minecraft", pathJarLibs);
	}

	async function downloadFileToCheerpJ() {
		const response = await fetch(urlDownloadMinecraft);
		const reader = response.body.getReader();
		const contentLength = +response.headers.get('Content-Length');

		const bytes = new Uint8Array(contentLength);
		progressBar.value = 0;
		progressBar.max = contentLength;

		let pos = 0;
		while (true) {
			const { done, value } = await reader.read();
			if (done) break;
			bytes.set(value, pos);
			pos += value.length;
			progressBar.value = pos;
		}

		// Write to CheerpJ filesystem
		return new Promise((resolve) => {
			var fds = [];
			cheerpOSOpen(fds, pathJarMinecraft, "w", fd => {
				cheerpOSWrite(fds, fd, bytes, 0, bytes.length, () => {
					cheerpOSClose(fds, fd, resolve);
				});
			});
		});
	}

	onMount(() => {
		loading = document.getElementById('loading');
		display = document.getElementById('display');
		intro = document.getElementById('intro');
		progressBar = document.getElementById('progress-bar');

		// Defer initialization to next animation frame to prevent stuck
		requestAnimationFrame(() => {
			startCheerpJ();
		});
	});
</script>

<style>
	/* Fullscreen foundation */
	html, body {
		height: 100%;
		width: 100%;
		margin: 0;
		padding: 0;
		overflow: hidden;
		font-family: system-ui, sans-serif;
		background: black;
	}

	@font-face {
		font-family: "Minecrafter";
		src: url("/fonts/minecrafter/Minecrafter.Reg.woff2"), sans-serif;
	}

	main {
		width: 100vw;
		height: 100vh;
		margin: 0;
		padding: 0;
		background: black;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.game-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		background: black;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		color: #eee;
		color-scheme: dark;
	}

	.display {
		display: flex;
		width: 100%;
		height: 100%;
		position: relative;
	}

	canvas {
		width: 100%;
		height: 100%;
	}

	.game-container > progress {
		width: calc(100% - 2em);
		margin: 1em;
	}

	.loading-container,
	.intro,
	.timeout-info,
	.timeout-timer {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		display: none;
		align-items: center;
		justify-content: center;
	}

	.timeout-info {
		background-color: rgba(0, 0, 0, 0.8);
		flex-direction: column;
	}

	.timeout-info > p {
		max-width: 60ch;
		text-align: center;
		margin: 0.75rem 0;
		color: white;
	}

	.timeout-info > h1 {
		font-size: 2rem;
		margin: 0;
		color: white;
	}

	.timeout-info > img {
		width: 80%;
		border: 1px solid white;
	}

	.timeout-timer > p {
		margin: 0 1rem 0 0;
		font-size: 4rem;
		color: white;
		text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
	}

	.spinner {
		width: 64px;
		height: 64px;
	}

	.nav-bar,
	.gh-container,
	.footer-container,
	.side-paragraph,
	.pagecontrols,
	.title,
	.main-container {
		display: none !important;
	}

	@media (max-width: 900px) {
		html, body {
			font-size: 0.9rem;
		}
	}
</style>

<div class="game-container">
	<div id="loading" class="loading-container" style="display:flex;">
		<img src={spinnerWhite} class="spinner" alt="Loading" />
		<p class="text-center">Loading CheerpJ ...</p>
	</div>
	<div id="intro" class="intro">
		<p>This is a proof-of-concept demo of Minecraft 1.2.5 running unmodified in the browser.</p>

		{#if !eulaAccepted}
			<p>
				<input type="checkbox" bind:checked={eulaAccepted} />
				Before playing, you have to accept the <a href="https://www.minecraft.net/eula" target="_blank">Minecraft EULA</a>
			</p>
		{/if}

		{#if eulaAccepted}
			<p>Clicking the button below will download the client from mojang.com.</p>
			<button on:click={startGame}>Play!</button>
		{/if}

		<div class="disclaimer">
			This is not an official Minecraft product. It is not approved by or associated with Mojang or Microsoft.
		</div>
	</div>
	<progress id="progress-bar"></progress>
	<div id="display" class="display"></div>
</div>
