<script lang="ts">
	import { onMount } from "svelte";

	import Vex from "vexflow";

	let canvas : HTMLElement;
	let width : number;
	let height : number;

	// FIXME: once start resizing the browser window, svelte enters a loop increasing canvas height

	onMount(() => {
		//const vf = Vex.Flow

		console.log("vf is", Vex, Vex.Flow)
		//console.log("vf flow is", vf)

		// Create a renderer with Canvas
		var renderer = new Vex.Flow.Renderer(
		    canvas,
		    Vex.Flow.Renderer.Backends.SVG
		);

		console.log('renderer', renderer)

		// Use the renderer to give the dimensions to the canvas
		renderer.resize(width, height);

		// Expose the context of the renderer
		const context = renderer.getContext();

		// And give some style to our canvas
		context.setFont("Arial", 10, "").setBackgroundFillStyle("#eed");

		/**
		 * Creating a new stave
		 */
		// Create a stave of width 400 at position x10, y40 on the canvas.
		const stave = new Vex.Flow.Stave(10, 40, 400);

		console.log('stave', stave)

		// Add a clef and time signature.
		stave.addClef("treble").addTimeSignature("4/4");
		// Set the context of the stave our previous exposed context and execute the method draw !
		stave.setContext(context).draw();


	});
</script>

<main>
	<h1>Impara a leggere la musica</h1>
	<div bind:clientWidth={width} bind:clientHeight={height}>
		<div class="music-score" bind:this={canvas}></div>
	</div>
	<div class="debug">
		<span>Canvas size: {width}&times;{height}</span>
	</div>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 1em;
		font-weight: 100;
	}

	.music-score {
		width: 100%;
		height: 200px;
	}

	.debug {
		color: darkgray;
		text-align: left;
		font-family: monospace;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>