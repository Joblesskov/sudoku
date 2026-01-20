<script>
	import { userGrid } from "@sudoku/stores/grid";
	import { cursor } from "@sudoku/stores/cursor";
	import { notes } from "@sudoku/stores/notes";
	import { candidates } from "@sudoku/stores/candidates";
	import { num } from "@sudoku/stores/currentValueStore";
	import { settings } from "@sudoku/stores/settings";

	// TODO: Improve keyboardDisabled
	import { keyboardDisabled } from "@sudoku/stores/keyboard";

	function handleKeyButton(number) {
		if (!$keyboardDisabled) {
			if ($notes) {
				if (number === 0) {
					candidates.clear($cursor);
				} else {
					candidates.add($cursor, number);
				}
				userGrid.set($cursor, 0);
			} else {
				userGrid.set($cursor, number);
				if ($num > 0) {
					userGrid.applyHint($cursor);
				}
			}
		}
	}

	function handleKey(e) {
		switch (e.key || e.keyCode) {
			case "ArrowUp":
			case 38:
			case "w":
			case 87:
				cursor.move(0, -1);
				break;

			case "ArrowDown":
			case 40:
			case "s":
			case 83:
				cursor.move(0, 1);
				break;

			case "ArrowLeft":
			case 37:
			case "a":
			case 65:
				cursor.move(-1);
				break;

			case "ArrowRight":
			case 39:
			case "d":
			case 68:
				cursor.move(1);
				break;

			case "Backspace":
			case 8:
			case "Delete":
			case 46:
				handleKeyButton(0);
				break;

			default:
				if (e.key && e.key * 1 >= 0 && e.key * 1 < 10) {
					handleKeyButton(e.key * 1);
				} else if (e.keyCode - 48 >= 0 && e.keyCode - 48 < 10) {
					handleKeyButton(e.keyCode - 48);
				}
				break;
		}
	}

	// 光标位于多候选数格子上
	$: isClickingCandidates = (() => {
		const cdkey = $cursor.x + "," + $cursor.y;
		const hintsLevel = $settings.hintsLevel;
		return (
			$candidates[cdkey] &&
			$candidates[cdkey].length > 1 &&
			hintsLevel >= $num &&
			$num >= $candidates[cdkey].length
		);
	})();

	function refreshDisableCandidate() {
		userGrid.disabledCandidates.set(new Map());
	}
</script>

<svelte:window
	on:keydown={handleKey}
/><!--on:beforeunload|preventDefault={e => e.returnValue = ''} />-->

<div class="keyboard-grid">
	{#each Array(10) as _, keyNum}
		{#if keyNum === 9}
			{#if isClickingCandidates}
				<button
					class="btn btn-key"
					disabled={$keyboardDisabled}
					title="Refresh Field"
					on:click={() => refreshDisableCandidate()}
				>
					<svg
						class="icon-outline"
						xmlns="http://www.w3.org/2000/svg"
						xmlns:xlink="http://www.w3.org/1999/xlink"
						version="1.1"
						viewBox="10 0 364 364"
						style="enable-background:new 0 0 512 512;"
						xml:space="preserve"
					>
						<g>
							<g>
								<path
									d="M182.595,325.678c-63.183,0-120.133-42.217-138.267-102.567c-2.833-9.067-12.183-14.167-21.25-11.333    c-9.067,2.833-14.167,12.183-11.333,21.25c22.95,75.933,91.517,126.65,170.85,126.65c98.317,0,178.5-80.183,178.5-178.5    s-80.183-178.5-178.5-178.5c-55.817,0-108.233,26.633-141.667,69.7l-7.083-56.1c-1.133-9.35-9.633-15.867-18.983-14.733    C5.511,2.678-1.005,11.178,0.128,20.528l13.317,103.7c1.133,8.5,8.5,14.733,16.717,14.733c0.567,0,1.417,0,1.983,0l102.567-11.617    c9.35-1.133,16.15-9.35,15.017-18.7s-9.35-16.15-18.7-15.017l-68.85,7.65c26.633-39.95,71.683-64.6,120.417-64.6    c79.617,0,144.5,64.883,144.5,144.5S262.211,325.678,182.595,325.678z"
								/>
							</g>
						</g>
					</svg>
				</button>
			{:else}
				<button
					class="btn btn-key"
					disabled={$keyboardDisabled}
					title="Erase Field"
					on:click={() => handleKeyButton(0)}
				>
					<svg
						class="icon-outline"
						xmlns="http://www.w3.org/2000/svg"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
						/>
					</svg>
				</button>
			{/if}
		{:else}
			<button
				class="btn btn-key"
				disabled={$keyboardDisabled}
				title="Insert {keyNum + 1}"
				on:click={() => handleKeyButton(keyNum + 1)}
			>
				{keyNum + 1}
			</button>
		{/if}
	{/each}
</div>

<style>
	.keyboard-grid {
		@apply grid grid-rows-2 grid-cols-5 gap-3;
	}

	.btn-key {
		@apply py-4 px-0;
	}
</style>
