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
		const key = (event.target as HTMLButtonElement).getAttribute(
			"data-key",
		);

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
</script>

<div id="screen">{currentGuess}</div>

<div class="controls">
	<div class="keyboard">
		<button
			onclick={update}
			data-key="backspace"
			name="key"
			value="backspace"
		>
			back
		</button>
		<button onclick={update} data-key="shift" name="key" value="shift">
			shift
		</button>
		<button
			onclick={update}
			data-key="caps lock"
			name="key"
			value="caps lock"
		>
			caps lock
		</button>

		{#each ["qwertyuiop", "asdfghjkl", "zxcvbnm", " "] as row (row)}
			<div class="row">
				{#each row as letter, index (index)}
					<button
						onclick={update}
						data-key={shift || capslock ? letter.toUpperCase() : letter}
						disabled={false}
						name="key"
						value={letter}
					>
						{shift || capslock ? letter.toUpperCase() : letter}
					</button>
				{/each}
			</div>
		{/each}
	</div>
</div>

<style>
	#screen {
		width: 500px;
		margin-left: auto;
		margin-right: auto;
		background-color: black;
		color: green;
		white-space: pre;
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

	.keyboard button,
	.keyboard button:disabled {
		--size: min(8vw, 4vh, 40px);
		background-color: white;
		color: black;
		width: var(--size);
		border: none;
		border-radius: 2px;
		font-size: calc(var(--size) * 0.5);
		margin: 0;
	}

	.keyboard button:focus {
		background: var(--color-theme-1);
		color: white;
		outline: none;
	}

	.keyboard button[data-key="shift"],
	.keyboard button[data-key="backspace"] {
		position: absolute;
		bottom: 0;
		width: calc(1.5 * var(--size));
		height: calc(1 / 3 * (100% - 2 * var(--gap)));
		text-transform: uppercase;
		font-size: calc(0.3 * var(--size));
		padding-top: calc(0.15 * var(--size));
	}

	.keyboard button[data-key="shift"] {
		right: calc(50% + 3.5 * var(--size) + 0.8rem);
	}

	.keyboard button[data-key=" "] {
		width: calc(var(--size) * 5);
	}
	.keyboard button[data-key="backspace"] {
		left: calc(50% + 3.5 * var(--size) + 0.8rem);
	}

	.keyboard button[data-key="enter"]:disabled {
		opacity: 0.5;
	}
</style>
