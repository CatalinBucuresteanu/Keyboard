<script lang="ts">
	import { enhance } from "$app/forms";
	import { resolve } from "$app/paths";
	import { confetti } from "@neoconfetti/svelte";
	import { MediaQuery } from "svelte/reactivity";

	import type { ActionData, PageData } from "./$types";

	interface Props {
		data: PageData;
		form: ActionData;
	}
	let {
		Letter_index,
		rotation = 10,
		width = 200,
		data,
		form = $bindable(),
	}: Props = $props();

	/** The current guess */
	let currentGuess = $state("");
	let shift = $state(false);
	let capslock = $state(false);
	let canType = $state(false);

	function update(letter: string) {
		var key = letter;

		if (key === "-") {
			key = "\u00A0";
		}

		if (key === "<") {
			currentGuess = currentGuess.slice(0, -1);
			if (form?.badGuess) form.badGuess = false;
		} else if (key === "shift") {
			shift = !shift;
		} else if (key === "caps lock") {
			capslock = !capslock;
		} else {
			currentGuess = currentGuess.concat(key);
			shift = false;
		}
	}

	let last_r = 0;
	let elem: HTMLButtonElement;

	function capturePointerMove(event: PointerEvent) {
		event.preventDefault();
		if (event.buttons !== 1) {
			last_r = 0;
			canType = false;
			return;
		}
		let rect = elem.getBoundingClientRect();
		let centerX = rect.left + rect.width / 2;
		let centerY = rect.top + rect.height / 2;
		let r = Math.sqrt(
			(event.clientX - centerX) * (event.clientX - centerX) +
				(event.clientY - centerY) * (event.clientY - centerY),
		);
		let theta = Math.atan2(
			event.clientY - centerY,
			event.clientX - centerX,
		);
		let Quadrant_size = (2 * Math.PI) / 40;
		if (theta < 0) theta += 2 * Math.PI;

		let delta_r = r - last_r;
		console.log(delta_r);
		if (delta_r < -10 && r < 50) {
			if (canType) {
				if (Letter_index < 28 && Letter_index >= 0) {
					update(alphabet[Letter_index]);
				} else if (Letter_index < 0) {
					Letter_index += 40;
				}
				canType = false;
			}
			last_r = r;
		} else if (delta_r > 0 || r > 50) {
			last_r = r;
			canType = true;
			Letter_index = Math.floor(theta / Quadrant_size);
			Letter_index = Letter_index - rotation;
		} else if (delta_r < 0 && canType == false) {
			last_r = r;
		}
	}

	let alphabet = "zyxwvutsrqponmlkjihgfedcba-<";
	function calcx(letter) {
		var letters = alphabet;
		var letter_index = letters.indexOf(letter) + rotation;
		var theta = ((2 * Math.PI) / 40) * (letter_index + 0.5);
		var x = width * 0.4 * Math.cos(theta);
		return x;
	}
	function calcy(letter) {
		var letters = alphabet;
		var letter_index = letters.indexOf(letter) + rotation;
		var theta = ((2 * Math.PI) / 40) * (letter_index + 0.5);
		var y = width * 0.4 * Math.sin(theta);
		return y;
	}
</script>

<p id="screen">{currentGuess}</p>

<div class="controls">
	<div class="keyboard">
		<button
			style="width: {width}pt; padding:0"
			class="circle"
			bind:this={elem}
			on:pointermove={capturePointerMove}
			on:pointerup={capturePointerMove}
		>
			{#each alphabet as item}
				<div
					class:highlight_effect={alphabet[Letter_index] === item && canType}
					style="transform: translate(-50%, -50%);position: absolute; left:{width *
						0.5 +
						calcx(item)}pt; top:{width * 0.5 + calcy(item)}pt;"
				>
					{item}
				</div>
			{/each}
			<div class="circle" style="width: 100pt; border: 2px dotted white;"></div>
		</button>
	</div>
</div>

<style>
	#screen {
		height: 50vh;
		width: 80%;
		max-width: 800px;
		margin-left: auto;
		margin-right: auto;
		background-color: black;
		color: green;
		overflow-wrap: break-word;
	}

	.controls {
		text-align: center;
		justify-content: center;
		height: min(18vh, 10rem);
	}

	.keyboard {
		--gap: 0.2rem;

		position: relative;
		display: flex;
		flex-direction: column;
		gap: var(--gap);
		height: 100%;
	}

	.keyboard .row {
		display: flex;
		justify-content: center;
		gap: 0.2rem;
		flex: 1;
	}

	.circle {
		position: relative;
		aspect-ratio: 1;
		touch-action: none;
		background-color: red;
		margin: auto;
		border-radius: 100%;
	}
	.highlight_effect {
		background-color: pink;
	}
</style>
