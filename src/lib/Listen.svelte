<script lang="ts">
	import { createEventDispatcher, onDestroy } from 'svelte';

	export let language = 'es-ES'; // en-US
	export let words: string[] = [];

	const dispatch = createEventDispatcher<{ text: string }>();

	let wakeLock: WakeLockSentinel;
	let permissions = false;
	let ready = false;
	let listening = false;
	let aborted = false;
	let text = '';

	let abort: () => void;

	$: grammar = `#JSGF V1.0; grammar words; public <word> =  ${words.join(' | ')};`;

	onDestroy(() => {
		abort?.();
		wakeLock?.release?.();
	});

	async function start() {
		try {
			await askForMedia();
			await askForScreenLock();
			permissions = true;
			listen();
		} catch (error) {
			console.log('...', error);
		}
		return () => {};
	}

	async function askForMedia() {
		console.log('Ask for media');
		await navigator.mediaDevices.getUserMedia({ audio: true });
	}

	async function askForScreenLock() {
		console.log('Ask for screen lock');
		try {
			wakeLock = await navigator.wakeLock.request('screen');
			wakeLock.addEventListener('release', () => {
				console.log('Wake Lock was released');
			});
			console.log('Wake Lock is active');
		} catch (err) {
			console.error(`${err.name}, ${err.message}`);
		}
	}

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
			dispatch('text', text);
		};

		recognition.onend = (e) => {
			console.log(e);
			!aborted && listen();
		};
		recognition.onaudiostart = () => (ready = true);
		recognition.onerror = (e) => console.log('onerror', e);
		recognition.onaudioend = console.log;
		recognition.onnomatch = console.log;
		recognition.onsoundend = console.log;
		recognition.onsoundstart = console.log;
		recognition.onspeechstart = () => (listening = true);
		recognition.onspeechend = () => (listening = false);
		// recognition.onstart = console.log;

		recognition.start();
		abort = () => {
			aborted = true;
			recognition.abort();
		};
	}
</script>

{#if !permissions}
	<div class="fixed bottom-0 flex w-full mb-10 justify-center items-center">
		<button class="w-20" on:click={start}>
			<img class="w-full" alt="start lo listen" src="/microphone.svg" />
		</button>
	</div>
{:else}
	<div class="fixed w-full left-0 bottom-0 text-center text-3xl">
		<div class:!bg-blue-500={listening} class:bg-red-500={!ready} class:bg-green-500={ready}>
			{text || 'Speak to start...'}
		</div>
	</div>
{/if}
