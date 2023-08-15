<script lang="ts">
	import Listen from '$lib/Listen.svelte';
	import { onMount } from 'svelte';

	const words = ['punto', 'siguiente', 'next', 'otro', '.'];
	const language = 'es-ES'; // en-US

	enum StichType {
		SINGLE = 'SINGLE',
		DOUBLE = 'DOUBLE',
		REDUCTION = 'REDUCTION'
	}

	let sound: HTMLAudioElement;
	let stepIndex = 0;
	let steps = [
		StichType.SINGLE,
		StichType.DOUBLE,
		StichType.SINGLE,
		StichType.DOUBLE,
		StichType.REDUCTION
	];

	$: displaySteps = [
		{ index: stepIndex - 1, step: steps[stepIndex - 1] },
		{ index: stepIndex, step: steps[stepIndex] },
		{ index: stepIndex + 1, step: steps[stepIndex + 1] }
	];

	onMount(() => {
		sound = new Audio('/pop.mp3');
	});

	function processText(text: string) {
		const found = words.some(
			(word) => word.toLocaleLowerCase().includes(text) || text.toLowerCase().includes(word)
		);
		if (found) {
			sound.play();
			stepIndex = stepIndex + 1;
		}
	}
</script>

<header class="sticky inset-0 bg-slate-800 p-2">
	<h1>PuntoPuntoPunto</h1>
</header>

<div class="m-20 grid grid-cols-3 text-center">
	{#each displaySteps as step (step.index)}
		{@const isLastStep = step.index === stepIndex - 1}
		{@const isCurrentStep = step.index === stepIndex}
		{@const isNextStep = step.index === stepIndex + 1}

		<div
			class="step transition-all duration-1000"
			class:font-bold={isCurrentStep}
			class:bg-green-600={isCurrentStep}
			class:bg-gray-600={isLastStep}
			class:text-gray-200={isLastStep}
			class:opacity-60={!isCurrentStep}
			class:bg-blue-500={isNextStep}
		>
			{#if step.step}
				{step.step}
			{/if}
		</div>
	{/each}
</div>

<Listen {words} {language} on:text={(e) => processText(e.detail)} />

<style>
	.step {
		animation: appear 250ms;
	}

	@keyframes appear {
		from {
			transform: translate(100px);
		}
		to {
			transform: translate(0px);
		}
	}
</style>
