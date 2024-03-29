<script>
	const test = ['hej', 'hejsan'];
</script>

<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

{#each test as thing}
	<h1>{thing}</h1>
	<p>weee</p>
{/each}
