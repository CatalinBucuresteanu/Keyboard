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
	let { data, form = $bindable() }: Props = $props();

	/** The current guess */
	let currentGuess = $state("");

	let shift = $state(false);
	let capslock = $state(false);

	function update(event: MouseEvent) {
		event.preventDefault();
		var key = (event.target as HTMLButtonElement).getAttribute(
			"data-key",
		);

		if (key === " ") {
			key = "\u00A0";
		}

		if (key === "backspace") {
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

	function click(event: MouseEvent) {
		event.preventDefault();
		let r=Math.sqrt((event.clientX*event.clientX)+(event.clientY*event.clientY))
		let theta=Math.atan2(event.clientY,event.clientX);
		console.log(r,theta);
	}
</script>

<p id="screen">{currentGuess}</p>

<div class="controls">
	<div class="keyboard">
	<button class="circle" onclick={click} >
a
	</button>

	
	</div>
</div>

<style>
	#screen {
		height: 50vh;
		width: 500px;
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

	.circle{
		width: 200pt;
		aspect-ratio: 1;
		background-color: red;
		margin: auto;
		border-radius: 100%;

	}
</style>
