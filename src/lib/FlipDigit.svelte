<script lang="ts">
	import { untrack } from 'svelte';
	let { value }: { value: string } = $props();
	let current = $state(untrack(() => value));
	let previous = $state(untrack(() => value));
	let running = $state(false);

	$effect(() => {
		const next = value;
		untrack(() => {
			if (next === current) return;
			previous = current;
			current = next;
			running = true;
		});
	});
</script>

<div class="flip-digit" aria-hidden="true">
	<div class="digit-half digit-top"><span>{current}</span></div>
	<div class="digit-half digit-bottom"><span>{running ? previous : current}</span></div>
	{#if running}
		{#key current}
			<div class="digit-half digit-top flap-out"><span>{previous}</span></div>
			<div class="digit-half digit-bottom flap-in" onanimationend={() => (running = false)}><span>{current}</span></div>
		{/key}
	{/if}
	<div class="hinge"></div>
</div>
