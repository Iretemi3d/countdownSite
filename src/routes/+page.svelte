<script lang="ts">
	import { onMount } from 'svelte';
	import FlipDigit from '#lib/FlipDigit.svelte';
	type TimeUnit = { label: string; value: string; color: string };
	const endTime = Date.parse('2026-11-08T12:00:00Z');
	let now = $state(Math.floor(Date.now() / 1_000) * 1_000);
	const pad = (value: number) => String(value).padStart(2, '0');
	let phase = $derived(now < endTime ? 'Time remaining' : 'Hackathon complete');
	let remaining = $derived(Math.max(0, endTime - now));
	let units: TimeUnit[] = $derived([
		{ label: 'Days', value: pad(Math.floor(remaining / 86_400_000)), color: '#D91114' },
		{ label: 'Hours', value: pad(Math.floor((remaining / 3_600_000) % 24)), color: '#EE320C' },
		{ label: 'Minutes', value: pad(Math.floor((remaining / 60_000) % 60)), color: '#F26E24' },
		{ label: 'Seconds', value: pad(Math.floor((remaining / 1_000) % 60)), color: '#FEC23C' }
	]);
	onMount(() => {
		const interval = window.setInterval(() => {
			now = Math.floor(Date.now() / 1_000) * 1_000;
		}, 1_000);
		return () => window.clearInterval(interval);
	});
</script>

<svelte:head><title>HackTheBurgh Countdown</title><meta name="description" content="A live flip-clock countdown to the end of HackTheBurgh." /></svelte:head>

<main>
	<section class="countdown-shell" aria-labelledby="page-title">
		<a class="brand-banner" href="https://hacktheburgh.com" target="_blank" rel="noreferrer" aria-label="Visit the HackTheBurgh website"><img src="/hacktheburgh-banner.svg" alt="HackTheBurgh" /></a>
		<h1 id="page-title" class="sr-only">HackTheBurgh Countdown</h1>
		<div class="clock" aria-live="polite" aria-label={`${phase}: ${units.map((unit) => `${unit.value} ${unit.label}`).join(', ')}`}>
			{#each units as unit}
				<div class="unit-wrap" style={`--accent: ${unit.color}`}>
					<FlipDigit value={unit.value} />
					<span class="unit-label">{unit.label}</span>
				</div>
			{/each}
		</div>

	</section>
</main>
