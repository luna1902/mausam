<script lang="ts">
	import type { HourWeather } from '$lib/types';
	import { dragscroll } from '@svelte-put/dragscroll';
	import WeatherIcon from './WeatherIcon.svelte';

	export let hourlyData: HourWeather[] = [];
	export let timeZone: string;
	let ref: HTMLDivElement;
	$: curIndex = 0;
	const hourToAmPm = (hours: number) => {
		var AmOrPm = hours >= 12 ? 'pm' : 'am';
		hours = hours % 12 || 12;
		return hours + AmOrPm;
	};
	$: hourlyData && scrollToCurIndex();

	const scrollToCurIndex = () => {
		curIndex = Number.parseInt(
			new Date().toLocaleTimeString('en-US', {
				timeZone,
				hour12: false,
				hour: 'numeric'
			})
		) % 24;
		if (ref) ref.scroll((ref.scrollWidth / 24) * curIndex, 0);
	};
</script>

<div
	bind:this={ref}
	use:dragscroll
	class="relative order-first col-span-2 flex h-48 cursor-grab select-none flex-row items-center justify-between gap-12 overflow-hidden overscroll-contain rounded-xl border bg-card p-6 text-card-foreground shadow-sm ring-offset-background transition-colors scrollbar-hide hover:overflow-x-auto focus:scroll-auto focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 md:order-2 md:p-6 lg:order-3"
>
	{#each hourlyData as item, index}
		<div class="flex h-full flex-col justify-between" 
		class:border={curIndex === index}
		class:font-bold={curIndex === index}>
			<div
				class="flex justify-center "
			>
				{hourToAmPm(index)}
			</div>
			<div class="flex h-full items-center justify-center">
				<WeatherIcon weatherCode={item.condition.code} isDay={1} class="h-8 w-8" />
			</div>
			<div class="flex justify-center">
				{Math.floor(item.temp_c)}&deg;
			</div>
		</div>
	{/each}
</div>