<script lang="ts">
	import { onMount } from 'svelte';
	import FlipDigit from '#lib/FlipDigit.svelte';
	type TimeUnit = { label: string; value: string; color: string };
	let now = $state(Date.now()); let endInput = $state('');
	let endTime = $state(0); let error = $state(''); let hasScheduled = $state(false);
	let showSetup = $state(true);
	const pad = (value: number) => String(value).padStart(2, '0');
	function toLocalInput(date: Date) { const offset = date.getTimezoneOffset() * 60_000; return new Date(date.getTime() - offset).toISOString().slice(0, 16); }
	function applySchedule() {
		const nextEnd = new Date(endInput).getTime();
		if (!nextEnd) { error = 'Choose an end time.'; return; }
		if (nextEnd <= Date.now()) { error = 'The end time needs to be in the future.'; return; }
		endTime = nextEnd; error = ''; hasScheduled = true;
		now = Date.now();
	}
	let phase = $derived(!hasScheduled ? 'Countdown not set' : now < endTime ? 'Time remaining' : 'Hackathon complete');
	let remaining = $derived(hasScheduled ? Math.max(0, endTime - now) : 0);
	let units: TimeUnit[] = $derived([
		{ label: 'Days', value: pad(Math.floor(remaining / 86_400_000)), color: '#D91114' },
		{ label: 'Hours', value: pad(Math.floor((remaining / 3_600_000) % 24)), color: '#EE320C' },
		{ label: 'Minutes', value: pad(Math.floor((remaining / 60_000) % 60)), color: '#F26E24' },
		{ label: 'Seconds', value: pad(Math.floor((remaining / 1_000) % 60)), color: '#FEC23C' }
	]);
	onMount(() => {
		const finish = new Date(Date.now() + 86_400_000); finish.setSeconds(0, 0);
		endInput = toLocalInput(finish);
		const interval = window.setInterval(() => {
			now = Date.now();
		}, 50);
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
		{#if showSetup}
			<form onsubmit={(event) => { event.preventDefault(); applySchedule(); }}>
				<div class="form-heading">
					<div><span class="step">Temporary setup</span><h2>Event timing</h2></div>
					<div class="form-actions">
						{#if hasScheduled}<span class="live-pill"><i></i> Live</span>{/if}
						<button class="dismiss-setup" type="button" onclick={() => (showSetup = false)} aria-label="Hide temporary setup until the page reloads" title="Hide until reload">×</button>
					</div>
				</div>
				<div class="fields"><label><span>Ends</span><input type="datetime-local" bind:value={endInput} required /></label><button type="submit">{hasScheduled ? 'Update clock' : 'Start countdown'}</button></div>
				{#if error}<p class="error" role="alert">{error}</p>{/if}
			</form>
		{/if}
	</section>
</main>
