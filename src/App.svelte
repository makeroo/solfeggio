<script lang="ts">
	import { onMount } from "svelte";

	import Vex from "vexflow";

	let canvas : HTMLElement;
	let width : number;
	let height : number;

	let renderer : Vex.Flow.Renderer

	const noteNamesIt = ['DO', 'RE', 'MI', 'FA', 'SOL', 'LA', 'SI'];
	const noteNamesInt = ['C', 'D', 'E', 'F', 'G', 'A', 'B'];
	let noteNames = noteNamesIt;

	let clef = 'treble';

	const availableNotes = {
		'treble': [
			'c/4',
			'd/4',
			'e/4',
			'f/4',
			'g/4',
			'a/4',
			'b/4',
			'c/5',
			'd/5',
			'e/5',
			'f/5',
			'g/5'
		],
		'bass': [
			'e/2',
			'f/2',
			'g/2',
			'a/2',
			'b/2',
			'c/3',
			'd/3',
			'e/3',
			'f/3',
			'g/3',
			'a/3',
			'b/3',
		]
	};
	let generatedNotes = [];

	onMount(() => {
		// Create a renderer with Canvas
		renderer = new Vex.Flow.Renderer(
		    canvas,
		    Vex.Flow.Renderer.Backends.SVG
		);

		render();

		generateNote();
	});

	function render() {
		// Use the renderer to give the dimensions to the canvas
		renderer.resize(width, height);

		// Expose the context of the renderer
		const context = renderer.getContext();

		context.clear()

		// And give some style to our canvas
		context.setFont("Arial", 10, "").setBackgroundFillStyle("#eed");

		/**
		 * Creating a new stave
		 */
		// Create a stave of width 400 at position x10, y40 on the canvas.
		const stave = new Vex.Flow.Stave(10, 40, 400);

		//console.log('stave', stave)

		// Add a clef and time signature.
		stave.addClef(clef).addTimeSignature("4/4");
		// Set the context of the stave our previous exposed context and execute the method draw !
		stave.setContext(context).draw();

		if (generatedNotes.length === 0) {
			return;
		}

		// TODO: generate random notes, one at a time
		const notes = generatedNotes.map(x => new Vex.Flow.StaveNote({clef: clef, keys: [x], duration: "q" }));

		// Create a voice in 4/4 and add above notes
		const voice = new Vex.Flow.Voice({num_beats: notes.length,  beat_value: 4});
		voice.addTickables(notes);

		// Format and justify the notes to 400 pixels.
		var formatter = new Vex.Flow.Formatter().joinVoices([voice]).format([voice], 400);

		// Render voice
		voice.draw(context, stave);
	}

	$: !canvas || reset(clef);

	function reset(required_to_trig_but_ignored: string) {
		generatedNotes = [];

		generateNote();
	}

	function answer (button: HTMLButtonElement, note: string) {
		console.log('selected note', note);

		const genNote = generatedNotes[generatedNotes.length - 1];
		let cls;

		if (genNote[0].toUpperCase() === note) {
			// yay!
			cls = 'right';
		} else {
			// error
			cls = 'error';
		}

		console.log(cls, 'expected', genNote, 'selected', note);

		button.className = `note ${cls}`;

		setTimeout(() => {
			button.className = 'note';

			generateNote();
		}, 1000)
	}

	function generateNote() {
		if (generatedNotes.length == 4) {
			generatedNotes = [];
		}

		const items = availableNotes[clef];
		const note = items[Math.floor(Math.random() * items.length)];

		generatedNotes.push(note);

		render();
	}
</script>

<main>
	<h1>Impara a leggere la musica</h1>
	<div class="settings">
		<div>
			<span>Chiave</span>
			<select bind:value={clef}>
				<option value="treble">SOL</option>
				<option value="bass">FA</option>
			</select>
		</div>
	</div>
	<div bind:clientWidth={width} bind:clientHeight={height}>
		<div class="music-score" bind:this={canvas}></div>
	</div>
	<div class="notes">
		{#each noteNames as note, i}
			<button type="button" class="note" on:click={(evt) => answer(evt.target, noteNamesInt[i])}>{note}</button>
		{/each}
	</div>
	<div class="debug">
		<span>Canvas size: {width}&times;{height}</span>
	</div>
</main>

<style>
	main {
		padding: 1em;
		max-width: 500px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 1em;
		font-weight: 100;
		text-align: center;
	}

	.music-score {
		width: 100%;
		height: 200px;
	}

	:global(.note) {
		margin: 2px 8px;
		width: 5em;
	}

	.debug {
		display: none;
		margin: 8px 0;
		color: darkgray;
		text-align: left;
		font-family: monospace;
	}

	:global(.error) {
		background-color: lightcoral;
	}

	:global(.right) {
		background-color: lightgreen;
	}
</style>