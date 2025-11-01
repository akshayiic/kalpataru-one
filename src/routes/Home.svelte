<script lang="js">
	import { getContext } from 'svelte';
	import { writable } from 'svelte/store';
	import 'lazysizes';
	import maximizeBtn from '$lib/images/maximize-icon.svg';
	import minimizeBtn from '$lib/images/minimize-icon.svg';
	import 'lazysizes/plugins/parent-fit/ls.parent-fit';
	import touchPoint1 from '$lib/images/touchPoint1.svg';
	const walkthroughDisabled = writable();
	$: walkthroughDisabled.set(false);

	const hotspotName = getContext('hotspotName');
	let currentUI = getContext('currentUI');
	import { goto } from '$app/navigation';
	const UIPanel = getContext('UIPanel');
	import { onMount } from 'svelte';

	const isDay = writable();
	const exteriorImagesSeq = writable();
	function toggleDayNight() {
		isDay.update((value) => !value);
		isDay.subscribe((value) => {
			hotspotName.set(value ? 'night' : 'Exterior');
		});
	}

	const isInteriorUnitDataMinimized = writable();
	$: isInteriorUnitDataMinimized.set(true);
	onMount(async () => {
		console.log(
			'%c[Hotspot Info]%c Current hotspotName:',
			'background: #222; color: #fff; padding: 2px 4px; border-radius: 2px;',
			'color: #00bfae; font-weight: bold;',
			$hotspotName
		);
		if ($hotspotName == 'Exterior') {
			// show('Exterior');
		}
		try {
			if (!document.querySelector('.exterior-cloudi')?.id) {
				console.log('init cloud');
				window.CI360.init();
			}
			// setTimeout(() => {
			// 	window.CI360.init();
			// }, 2000);
		} catch (error) {
			console.error('Error initializing cloud:', error);
		}
	});
</script>

{#if !$currentUI.highlights && false}
	<button
		id="daynighmode"
		type="button"
		class="day-night-toggle"
		on:click={toggleDayNight}
		style="position: fixed; top: 3rem; right: 2rem; cursor: pointer; z-index: 1000001; background: white; padding: .4rem; border-radius: 50%;"
		aria-label="Toggle day and night mode"
	>
		{#if $isDay}
			<!-- Replace with lucide Sun icon -->
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="icon icon-sun"
				width="24"
				height="24"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
			>
				<circle cx="12" cy="12" r="5"></circle>
				<line x1="12" y1="1" x2="12" y2="3"></line>
				<line x1="12" y1="21" x2="12" y2="23"></line>
				<line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
				<line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
				<line x1="1" y1="12" x2="3" y2="12"></line>
				<line x1="21" y1="12" x2="23" y2="12"></line>
				<line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
				<line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
			</svg>
		{:else}
			<!-- Replace with lucide Moon icon -->
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="icon icon-moon"
				width="24"
				height="24"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
			>
				<path d="M21 12.79A9 9 0 0111.21 3 7 7 0 1012 21a9 9 0 009-8.21z"></path>
			</svg>
		{/if}
	</button>
{/if}
<div
	class={$hotspotName == 'Exterior'
		? 'cloudimage-360 exterior-cloudi !absolute left-0 top-0  z-[10]'
		: 'cloudimage-360 invisible'}
	data-folder="https://assets.vestate.io/kalpataru-one/exterior/"
	data-filename="{'ext{'}index{'}'}.webp"
	data-amount="24"
	data-keys="false"
	data-filters="blur:20"
	data-drag-speed="400"
	data-request-responsive-images="true"
	data-info="false"
	data-ratio=" 1"
></div>

<div
	class={$hotspotName == 'night'
		? 'cloudimage-360 exterior-cloudi !absolute left-0 top-0  z-[10]'
		: 'cloudimage-360 invisible'}
	data-folder="https://assets.vestate.io/runwal-kane/exterior/night/"
	data-filename="{'night{'}index{'}'}.webp"
	data-amount="24"
	data-keys="false"
	data-filters="blur:20"
	data-drag-speed="400"
	data-request-responsive-images="true"
	data-info="false"
	data-ratio="1"
></div>

<style>
	@keyframes fadeIn {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}
	@keyframes fadeOut {
		from {
			opacity: 1;
		}
		to {
			opacity: 0;
		}
	}
	.touch-point-content {
		transition: all 1s ease-in-out;
		animation: fadeOut 0.7s forwards;
		visibility: hidden;
	}
	.touch-point-wrapper {
		position: absolute;
		top: 65%;
		z-index: 99999;
		right: 51%;
		cursor: pointer;
		display: flex;
		align-items: center;
	}

	.touch-point-wrapper-3 {
		position: absolute;
		bottom: 18%;
		right: 56.3%;
		display: flex;
		cursor: pointer;
		align-content: center;
		align-items: center;
		gap: 0.6rem;
	}
</style>
