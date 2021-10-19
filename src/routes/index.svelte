<script>
	import { fly, fade } from 'svelte/transition';
	import { quintInOut } from 'svelte/easing';
	import Button from '../components/ui/buttons/Button.svelte';

	let pokemons = null;
	let limit = 20;
	let delay = 2000;

	// button states
	let isLoading = false;
	let isError = false;
	let isDone = false;

	async function fetchPokemon() {
		const response = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=${limit}`);
		return response.json();
	}

	function handleButton() {
		if (!pokemons) {
			pokemons = fetchPokemon();
			// set loading state
			isLoading = true;
			// reset loading state
			setTimeout(() => {
				isLoading = false;
				isDone = true;
				// return to default
				setTimeout(() => {
					isDone = false;
				}, delay);
			}, delay);
		} else {
			// if I've already fetched then
			// switch to error state
			isError = true;
			setTimeout(() => {
				isError = false;
			}, delay);
		}
	}
</script>

<div class="flex my-8 justify-center">
	<Button
		on:click={handleButton}
		loading={isLoading}
		error={isError}
		done={isDone}
		loadingClass="bg-yellow-600 scale-110 active:bg-yellow-600"
		errorClass="bg-red-600 scale-110 shake active:bg-red-600"
		doneClass="bg-green-600 scale-110 active:bg-green-600"
	>
		{#if isLoading}
			Catching Pokemons...
		{:else if isError}
			You've already caught 'em all
		{:else if isDone}
			Got 'em!
		{:else}
			Catch Pokemon
		{/if}
	</Button>
</div>

{#if pokemons}
	<div
		transition:fade={{ duration: 800, easing: quintInOut }}
		class="grid grid-cols-2 lg:grid-cols-5 gap-8 my-8"
	>
		{#await pokemons}
			Loading...
		{:then result}
			{#each result.results as pokemon, i}
				<div
					class="border border-gray-600 p-8 rounded-xl text-white bg-gray-800 hover:bg-gray-900 shadow-lg capitalize"
					transition:fly={{ duration: 200, y: 30, delay: i * 100 }}
				>
					<img
						src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/{i +
							1}.png"
						alt={pokemon.name}
					/>
					<h3 class="text-2xl font-extrabold">{pokemon.name}</h3>
					<h5 class="text-base">Pokemon #{i + 1}</h5>
				</div>
			{/each}
		{:catch error}
			An error has occurred {error}
		{/await}
	</div>
{/if}
