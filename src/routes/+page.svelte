<script lang="ts">
	import { onMount } from 'svelte';
	type TimeUnit = { label: string; value: string; color: string };
	let now = $state(Date.now()); let startInput = $state(''); let endInput = $state('');
	let startTime = $state(0); let endTime = $state(0); let error = $state(''); let hasScheduled = $state(false);
	let previousValues = $state(['00', '00', '00', '00']);
	let lastValues = ['00', '00', '00', '00'];
	const pad = (value: number) => String(value).padStart(2, '0');
	function valuesAt(timestamp: number) {
		const target = timestamp < startTime ? startTime : endTime;
		const left = hasScheduled ? Math.max(0, target - timestamp) : 0;
		return [pad(Math.floor(left / 86_400_000)), pad(Math.floor((left / 3_600_000) % 24)), pad(Math.floor((left / 60_000) % 60)), pad(Math.floor((left / 1_000) % 60))];
	}
	function toLocalInput(date: Date) { const offset = date.getTimezoneOffset() * 60_000; return new Date(date.getTime() - offset).toISOString().slice(0, 16); }
	function applySchedule() {
		const nextStart = new Date(startInput).getTime(); const nextEnd = new Date(endInput).getTime();
		if (!nextStart || !nextEnd) { error = 'Choose both a start and end time.'; return; }
		if (nextEnd <= nextStart) { error = 'The end time needs to be after the start time.'; return; }
		previousValues = units.map((unit) => unit.value);
		startTime = nextStart; endTime = nextEnd; error = ''; hasScheduled = true;
		lastValues = valuesAt(Date.now());
	}
	let phase = $derived(now < startTime ? 'Hackathon begins in' : now < endTime ? 'Time remaining' : 'Hackathon complete');
	let targetTime = $derived(now < startTime ? startTime : endTime);
	let remaining = $derived(hasScheduled ? Math.max(0, targetTime - now) : 0);
	let units: TimeUnit[] = $derived([
		{ label: 'Days', value: pad(Math.floor(remaining / 86_400_000)), color: '#D91114' },
		{ label: 'Hours', value: pad(Math.floor((remaining / 3_600_000) % 24)), color: '#EE320C' },
		{ label: 'Minutes', value: pad(Math.floor((remaining / 60_000) % 60)), color: '#F26E24' },
		{ label: 'Seconds', value: pad(Math.floor((remaining / 1_000) % 60)), color: '#FEC23C' }
	]);
	onMount(() => {
		const base = new Date(); base.setMinutes(base.getMinutes() + 5, 0, 0); const finish = new Date(base.getTime() + 86_400_000);
		startInput = toLocalInput(base); endInput = toLocalInput(finish);
		const interval = window.setInterval(() => {
			const nextNow = Date.now();
			const nextValues = valuesAt(nextNow);
			if (nextValues.some((value, index) => value !== lastValues[index])) {
				previousValues = [...lastValues];
				lastValues = nextValues;
			}
			now = nextNow;
		}, 50);
		return () => window.clearInterval(interval);
	});
</script>

<svelte:head><title>Hackathon Countdown</title><meta name="description" content="A live flip-clock countdown for hackathon start and finish times." /></svelte:head>

<main>
	<div class="ambient ambient-one"></div><div class="ambient ambient-two"></div>
	<section class="countdown-shell" aria-labelledby="page-title">
		<a class="brand-banner" href="https://hacktheburgh.com" target="_blank" rel="noreferrer" aria-label="Visit the HackTheBurgh website"><img src="/hacktheburgh-banner.svg" alt="HackTheBurgh" /></a>
		<header><h1 id="page-title">HackTheBurgh Countdown</h1></header>
		<div class="clock" aria-live="polite" aria-label={`${phase}: ${units.map((unit) => `${unit.value} ${unit.label}`).join(', ')}`}>
			{#each units as unit, index}<div class="unit-wrap">{#key unit.value}<div class="flip-card" style={`--accent: ${unit.color}`}>
				<div class="clock-half clock-top"><span>{unit.value}</span></div>
				<div class="clock-half clock-bottom"><span>{unit.value}</span></div>
				<div class="clock-half bottom-old" aria-hidden="true"><span>{previousValues[index]}</span></div>
				<div class="clock-half flap-out" aria-hidden="true"><span>{previousValues[index]}</span></div>
				<div class="clock-half flap-in" aria-hidden="true"><span>{unit.value}</span></div>
				<div class="hinge"></div>
			</div>{/key}<span class="unit-label">{unit.label}</span></div>{/each}
		</div>
		<form onsubmit={(event) => { event.preventDefault(); applySchedule(); }}>
			<div class="form-heading"><div><span class="step">Temporary setup</span><h2>Event timing</h2></div>{#if hasScheduled}<span class="live-pill"><i></i> Live</span>{/if}</div>
			<div class="fields"><label><span>Starts</span><input type="datetime-local" bind:value={startInput} required /></label><label><span>Ends</span><input type="datetime-local" bind:value={endInput} required /></label><button type="submit">{hasScheduled ? 'Update clock' : 'Start countdown'}</button></div>
			{#if error}<p class="error" role="alert">{error}</p>{/if}
		</form>
	</section>
</main>
