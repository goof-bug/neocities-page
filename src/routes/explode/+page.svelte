<script lang="ts">
	import GIF from 'gif.js';
	import { frames } from '$lib/explodeFrames/frames';

	const gifWorkerUrl = new URL('gif.js/dist/gif.worker.js', import.meta.url);

	let files: FileList | undefined = $state();

	let gifPromise = $derived.by(async () => {
		if (files && files.length) {
			const temporaryObjectUrls = [];
			try {
				const gif = new GIF({
					width: 498,
					height: 460,
					debug: true,
					workerScript: gifWorkerUrl.toString(),
					quality: 1
				});
				for (const file of files) {
					const img = new Image();
					const temporaryObjectUrl = URL.createObjectURL(file);
					temporaryObjectUrls.push(temporaryObjectUrl);
					img.src = temporaryObjectUrl;
					await img.decode();
					gif.addFrame(img, { delay: 1000 });
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
			} finally {
				for (const temporaryObjectUrl of temporaryObjectUrls) {
					URL.revokeObjectURL(temporaryObjectUrl);
				}
			}
		}
	});
</script>

<div class="container">
	<h1>Explode</h1>
	<input type="file" bind:files accept="image/*" />

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
