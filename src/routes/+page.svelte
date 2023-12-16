<script lang="ts">
	import ColorSelect, {type RGB} from "svelte-color-select";
	import ColorPicker from "svelte-awesome-color-picker";

	let colorPickerIndex = 0;
	let isFractionalRgb = true;
	let rgb: RGB = {r: 0, g: 0, b: 0};
	let cssValue = getCurrentColorAsCss();

	let hasHiddenManually = false;
	let helpHidden = false;

	let posX = 0, posY = 0;

	function onGlobalKeyDown(event) {
		switch (event.key) {
			case "h":
				hide(false);
				break;
			case "F11":
				hide(true);
				break;
			default:
				break;
		}
	}

	function onMouseUp(event) {
		function togglePicker() {
			event.preventDefault();

			posX = event.clientX;
			posY = event.clientY;
		}

		if (event.button === 0) { // Left Click
			colorPickerIndex = colorPickerIndex === 1 ? 0 : 1;
			togglePicker();
		} else if (event.button === 2) { // Right Click
			colorPickerIndex = colorPickerIndex === 2 ? 0 : 2;
			togglePicker();
		}
	}

	function disableEvent(event) {
		event.stopPropagation();
		event.preventDefault();
	}

	function hide(automatic: boolean) {
		if (automatic) {
			if (!helpHidden) {
				helpHidden = true;
			} else if (helpHidden && !hasHiddenManually) {
				helpHidden = false;
			}

			return;
		}

		helpHidden = !helpHidden;
		hasHiddenManually = true;
	}

	function getCurrentColorAsCss() {
		const mul = isFractionalRgb ? 255 : 1;
		return `rgb(${rgb.r * mul}, ${rgb.g * mul}, ${rgb.b * mul})`;
	}

	function toggleFullscreen() {
		hide(true);

		if (document.fullscreenElement) {
			document.exitFullscreen();
		}

		const elem = document.documentElement;

		if (elem.requestFullscreen) {
			elem.requestFullscreen();
		} else if (elem.webkitRequestFullscreen) { /* Safari */
			elem.webkitRequestFullscreen();
		} else if (elem.msRequestFullscreen) { /* IE11 */
			elem.msRequestFullscreen();
		}
	}
</script>

<svelte:window on:keydown={onGlobalKeyDown} on:contextmenu={disableEvent}/>

<div class="backgroundDiv" style:background-color={cssValue} on:mouseup={onMouseUp}>
</div>

{#if !helpHidden}
	<div class="helpText">
		<a href="https://github.com/Ceiridge/Fullscreen-Color-Picker" target="_blank"><h1>Fullscreen Color Picker</h1>
		</a>

		<ul>
			<li><b>F11</b>: Enter fullscreen or <a href="#" on:click={toggleFullscreen}>click here</a></li>
			<li><b>H</b>: Hide/Show this help explainer</li>
			<li><b>Left Click</b>: Open OKHsv color picker</li>
			<li><b>Right Click</b>: Open HSV color picker</li>
		</ul>
	</div>
{/if}

{#if colorPickerIndex !== 0}
	<div class="colorPicker" style:left={posX.toFixed(2) + "px"} style:top={posY.toFixed(2) + "px"}>
		{#if colorPickerIndex === 1}
			<ColorSelect bind:rgb on:change={() => {isFractionalRgb = true; cssValue = getCurrentColorAsCss();}}/>
		{:else if colorPickerIndex === 2}
			<ColorPicker bind:rgb on:input={() => {isFractionalRgb = false; cssValue = getCurrentColorAsCss();}}
						 isA11y={false} isOpen={true} disableCloseClickOutside={true} label="" isInput={false}/>
		{/if}
	</div>
{/if}