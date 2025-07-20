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

		startCheerpJ();
	});
</script>

<style>
	html, body {
		margin: 0;
		padding: 0;
		width: 100vw;
		height: 100vh;
		overflow: hidden;
		font-family: sans-serif;
		background: black;
		color: white;
	}

	.game-container {
		width: 100vw;
		height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		position: relative;
	}

	#display {
		width: 100%;
		height: 100%;
		position: absolute;
		top: 0;
		left: 0;
		z-index: 1;
		display: none;
	}

	#display.show {
		display: block;
	}

	.loading-container,
	.intro {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		z-index: 2;
		text-align: center;
		background: rgba(0, 0, 0, 0.8);
		padding: 2rem;
		border-radius: 1rem;
	}

	progress {
		position: absolute;
		bottom: 2rem;
		width: 80%;
		max-width: 600px;
		z-index: 2;
	}

	button {
		padding: 0.5rem 1rem;
		font-size: 1rem;
		cursor: pointer;
		margin-top: 1rem;
	}

	.spinner {
		width: 3rem;
		height: 3rem;
		margin-bottom: 1rem;
	}

	a {
		color: #00aaff;
	}

	.disclaimer {
		font-size: 0.8rem;
		margin-top: 1.5rem;
		color: #ccc;
	}
</style>

<div class="game-container">
	<div id="loading" class="loading-container">
		<img src={spinnerWhite} class="spinner" alt="Loading" />
		<p>Loading CheerpJ ...</p>
	</div>

	<div id="intro" class="intro">
		<p>This is a proof-of-concept demo of Minecraft 1.2.5 running unmodified in the browser.</p>

		{#if !eulaAccepted}
			<p>
				<input type="checkbox" bind:checked={eulaAccepted} />
				Before playing, accept the <a href="https://www.minecraft.net/eula" target="_blank">Minecraft EULA</a>
			</p>
		{/if}

		{#if eulaAccepted}
			<p>Click the button below to download the client from mojang.com.</p>
			<button on:click={startGame}>Play!</button>
		{/if}

		<div class="disclaimer">
			This is not an official Minecraft product. It is not approved by or associated with Mojang or Microsoft.
		</div>
	</div>

	<progress id="progress-bar"></progress>
	<div id="display" class="display"></div>
</div>
