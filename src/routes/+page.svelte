<script lang="ts">
	import { onMount } from 'svelte';

	const language = 'es-ES'; // en-US
	const words = ['punto', 'siguiente', 'next', 'otro'];
	const grammar = `#JSGF V1.0; grammar words; public <word> =  ${words.join(' | ')};`;

	enum StichType {
		SINGLE = 'SINGLE',
		DOUBLE = 'DOUBLE',
		REDUCTION = 'REDUCTION'
	}

	let sound: HTMLAudioElement;
	let listening = false;
	let ready = false;
	let text = '';
	let aborted = false;

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
		aborted = false;
		const abort = listen();
		return () => abort();
	});

	function listen() {
		console.log('start...');
		const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
		const SpeechGrammarList = window.SpeechGrammarList || window.webkitSpeechGrammarList;

		const recognition = new SpeechRecognition();
		const speechRecognitionList = new SpeechGrammarList();
		speechRecognitionList.addFromString(grammar, 1);
		recognition.grammars = speechRecognitionList;
		recognition.continuous = false;
		recognition.lang = language;
		recognition.maxAlternatives;

		recognition.onresult = (event) => {
			console.log(event);
			text = event.results[event.results.length - 1][0].transcript;
			ready = false;
			processText(text);
		};

		recognition.onend = () => !aborted && listen();
		recognition.onaudiostart = () => (ready = true);
		recognition.onerror = console.log;
		recognition.onaudioend = console.log;
		recognition.onnomatch = console.log;
		recognition.onsoundend = console.log;
		recognition.onsoundstart = console.log;
		recognition.onspeechstart = () => (listening = true);
		recognition.onspeechend = () => (listening = false);
		recognition.onstart = console.log;

		recognition.start();
		return () => {
			aborted = true;
			recognition.abort();
		};
	}

	function processText(text: string) {
		const found = words.some(
			(word) => word.toLocaleLowerCase().includes(text) || text.toLowerCase().includes(word)
		);
		console.log(text, '??', found);
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

<div class="fixed w-full left-0 bottom-0 text-center text-3xl">
	<div class:!bg-blue-500={listening} class:bg-red-500={!ready} class:bg-green-500={ready}>
		{text || 'Speak to start...'}
	</div>
</div>

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
