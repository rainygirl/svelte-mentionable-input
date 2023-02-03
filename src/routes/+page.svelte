<script lang="ts">
	import CJKMentionableInput from './CJKMentionableInput.svelte';

	let textValue: string | undefined;
</script>

<svelte:head>
	<title>MentionableInput</title>
</svelte:head>

<section>
	<h2>Enter letters including # or @</h2>

	<CJKMentionableInput bind:value={textValue} />

	<h2>Value:</h2>
	<textarea bind:value={textValue} />
</section>

<style>
	textarea {
		width: 100%;
		min-height: 5em;
	}
	section {
		font-size: 1em;
	}
</style>
