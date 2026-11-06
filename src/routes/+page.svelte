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
	<svg class="grid-path" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
		<defs>
			<linearGradient id="route-gradient" x1="0" y1="0" x2="100" y2="100" gradientUnits="userSpaceOnUse">
				<stop offset="0" stop-color="#D91114" />
				<stop offset=".34" stop-color="#EE320C" />
				<stop offset=".68" stop-color="#F26E24" />
				<stop offset="1" stop-color="#FEC23C" />
			</linearGradient>
			<g id="junction-dot">
				<path class="junction-ring" d="M0 0h.01" />
				<path class="junction-core" d="M0 0h.01" />
			</g>
		</defs>
		<g class="route-map">
		<path class="route" d="M0 33.333 H14 Q15 33.333 15 31.667 V6.667 Q15 5 16 5 H34 Q35 5 35 6.667 V31.667 Q35 33.333 36 33.333 H69 Q70 33.333 70 31.667 V18.333 Q70 16.667 71 16.667 H84 Q85 16.667 85 18.333 V31.667 Q85 33.333 86 33.333 H100" />
		<path class="route route-bottom" d="M0 83.333 H39 Q40 83.333 40 81.667 V76.667 Q40 75 41 75 H59 Q60 75 60 76.667 V81.667 Q60 83.333 61 83.333 H79 Q80 83.333 80 81.667 V76.667 Q80 75 81 75 H100" />
		<g class="junctions">
			<use href="#junction-dot" x="15" y="5" /><use href="#junction-dot" x="35" y="5" />
			<use href="#junction-dot" x="15" y="33.333" /><use href="#junction-dot" x="35" y="33.333" />
			<use href="#junction-dot" x="70" y="33.333" /><use href="#junction-dot" x="70" y="16.667" />
			<use href="#junction-dot" x="85" y="16.667" /><use href="#junction-dot" x="85" y="33.333" />
			<use href="#junction-dot" x="40" y="83.333" /><use href="#junction-dot" x="40" y="75" />
			<use href="#junction-dot" x="60" y="75" /><use href="#junction-dot" x="60" y="83.333" />
			<use href="#junction-dot" x="80" y="83.333" /><use href="#junction-dot" x="80" y="75" />
		</g>
		</g>
	</svg>
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
