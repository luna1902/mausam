<script lang="ts">
	import { PUBLIC_GOOGLE_MAP_API_KEY } from '$env/static/public';
	import { DEFAULT_SUGGESTIONS } from '$lib/constants';
	import { encode, loadGooglePlacesLibrary } from '$lib/utils';
	import { shortcut, type ShortcutEventDetail } from '@svelte-put/shortcut';
	import debounce from 'lodash.debounce';
	import { onMount } from 'svelte';
	import { MagnifyingGlassSolid } from 'svelte-awesome-icons';
	import { createDialog } from 'svelte-headlessui';
	import Transition from 'svelte-transition';

	const dialog = createDialog({ label: 'Search City...' });

	let autocomplete: any;
	let googleLibraryLoaded = false;
	let searchStr = '';
	let suggestions: any[] = DEFAULT_SUGGESTIONS;
	let searchRef: HTMLInputElement;

	$: if ($dialog.expanded) {
		searchRef.focus();
	}

	const handleInput = debounce(async (event: Event) => {
		const target = event.target as HTMLInputElement;
		searchStr = target.value;
		if (searchStr == '') {
			suggestions = DEFAULT_SUGGESTIONS;
		} else {
			if (googleLibraryLoaded) {
				autocomplete.getPlacePredictions(
					{ input: target.value, types: ['(cities)'] },
					(e: any[]) => {
						suggestions = e;
					}
				);
			}
		}
	}, 300);

	const onShortcut = (event: ShortcutEventDetail) => {
		const keyboardEvent = event.originalEvent;
		keyboardEvent.preventDefault();
		dialog.open();
	};

	const onSelectCity = () => {
		searchRef.value = '';
		searchStr = '';
		dialog.close();
	};

	onMount(() => {
		loadGooglePlacesLibrary(PUBLIC_GOOGLE_MAP_API_KEY, async () => {
			const { AutocompleteService } = await google.maps.importLibrary('places');
			autocomplete = new AutocompleteService();
			googleLibraryLoaded = true;
		});
	});
</script>

<svelte:window
	use:shortcut={{
		trigger: {
			key: 'j',
			modifier: ['ctrl', 'meta'],
			callback: onShortcut
		}
	}}
/>

<button
	on:click={dialog.open}
	class="inline-flex h-9 w-full items-center justify-center whitespace-nowrap rounded-md border border-input bg-background px-4 text-sm font-medium ring-offset-background transition-colors hover:bg-accent/50 hover:text-accent-foreground focus:ring-0 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50"
>
	<p class="text-sm text-muted-foreground">
		Search city...{' '}
		<kbd
			class="pointer-events-none ml-auto inline-flex h-5 select-none items-center gap-1 rounded border bg-muted px-1.5 font-mono text-xs font-medium text-muted-foreground opacity-100 hover:bg-primary md:ml-28"
		>
			<span class="text-xs">⌘</span>J
		</kbd>
	</p>
</button>

<Transition show={$dialog.expanded}>
	<Transition
		enter="ease-out duration-300"
		enterFrom="opacity-0"
		enterTo="opacity-100"
		leave="ease-in duration-200"
		leaveFrom="opacity-100"
		leaveTo="opacity-0"
	>
		<button
			class="fixed inset-0 z-10 bg-background/80 bg-opacity-25 backdrop-blur-sm"
			on:click={dialog.close}
		/>
	</Transition>

	<div class="fixed inset-0 z-20 overflow-y-auto">
		<div class="flex min-h-full items-center justify-center p-4 text-center">
			<Transition
				enter="ease-out duration-300"
				enterFrom="opacity-0 scale-95"
				enterTo="opacity-100 scale-100"
				leave="ease-in duration-200"
				leaveFrom="opacity-100 scale-100"
				leaveTo="opacity-0 scale-95"
			>
				<div
					class="grid w-full max-w-md transform overflow-hidden rounded-2xl border bg-background text-left align-middle shadow-lg transition-all duration-200"
					use:dialog.modal
				>
					<div class="flex items-center border-b px-3">
						<MagnifyingGlassSolid class="mr-2 h-4 w-4 shrink-0 opacity-50" />
						<input
							bind:this={searchRef}
							on:input={handleInput}
							placeholder="Search city..."
							class="flex h-11 w-full rounded-md bg-transparent py-3 text-sm outline-none placeholder:text-muted-foreground disabled:cursor-not-allowed disabled:opacity-50"
						/>
					</div>
					<div class="max-h-[300px] overflow-y-auto overflow-x-hidden">
						{#if suggestions.length > 0}
							<div class="overflow-hidden p-1 text-foreground">
								<p class="px-2 py-1.5 text-xs font-medium text-muted-foreground">Suggestions</p>
								{#each suggestions as suggestion}
									<a
										href={`/${encode(suggestion.description || suggestion)}`}
										on:click={onSelectCity}
										class="relative flex w-full cursor-pointer select-none items-center rounded-sm px-2 py-3 text-sm outline-none hover:bg-accent hover:text-accent-foreground"
									>
										{suggestion.description || suggestion}
									</a>
								{/each}
							</div>
						{:else}
							<div class="py-6 text-center text-sm">No results found.</div>
						{/if}
					</div>

					<button
						on:click={dialog.close}
						class="absolute right-3 top-3 rounded-sm opacity-70 ring-offset-background transition-opacity hover:opacity-100 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 disabled:pointer-events-none"
					>
						<kbd
							class="pointer-events-none flex h-6 select-none items-center justify-center rounded-md border bg-transparent px-1.5 py-1 font-mono font-medium tracking-tight text-muted-foreground opacity-100"
						>
							<span class="text-xs">Esc</span>
						</kbd>
						<span class="sr-only">Close</span>
					</button>
				</div>
			</Transition>
		</div>
	</div>
</Transition>