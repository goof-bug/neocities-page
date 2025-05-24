<script lang="ts">
	import GIF from 'gif.js';
	import { frames } from '$lib/explodeFrames/frames';

	const gifWorkerUrl = new URL('gif.js/dist/gif.worker.js', import.meta.url);

	let canvas: HTMLCanvasElement | undefined;

	let files: FileList | undefined = $state();

	let gifPromise = $derived.by(async () => {
		if (files && files.length && canvas) {
			const ctx = canvas.getContext('2d');
			if (!ctx) {
				throw new Error('No canvas context');
			}

			const workerRequest = await fetch(gifWorkerUrl);
			const workerBlob = await workerRequest.blob();

			const gif = new GIF({
				width: 498,
				height: 460,
				debug: true,
				workerScript: URL.createObjectURL(workerBlob),
				quality: 1
			});
			for (const file of files) {
				const image = await window.createImageBitmap(file);
				ctx.drawImage(image, 0, 0, 498, 460);
				gif.addFrame(ctx, { delay: 1000, copy: true });
			}
			for (const frame of frames) {
				const img = new Image();
				img.src = frame;
				await img.decode();
				gif.addFrame(img, { delay: 10 });
			}
			const { promise, resolve, reject } = Promise.withResolvers<Blob>();
			gif.on('finished', (blob) => {
				resolve(blob);
			});
			gif.on('abort', () => {
				reject(new Error('Aborted'));
			});

			gif.render();

			const blob = await promise;
			return URL.createObjectURL(blob);
		}
	});
</script>

<div class="container">
	<h1>Explode him</h1>
	<input type="file" bind:files accept="image/*" />

	<canvas class="hidden" bind:this={canvas} width="498" height="460"></canvas>

	{#await gifPromise}
		<p>Loading...</p>
	{:then gif}
		{#if !gif}
			<p>No files selected.</p>
			<p>Select an image file.</p>
		{:else}
			<img src={gif} alt="Exploded" />
			<a href={gif} download="exploded.gif">Download</a>
		{/if}
	{:catch error}
		<p>Error: {error}</p>
	{/await}
</div>

<style>
	.hidden {
		display: none;
	}

	.container {
		display: flex;
		flex-direction: column;
		gap: 2em;
	}

	@media (min-width: 1200px) {
		.container {
			width: 40%;
			margin: 0 auto;
		}
	}
</style>
