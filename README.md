**Perceived performance** is the illusion of speed we experience when a website is really good at letting us know exactly what's happening at all times.

Today I'm going to show you how to create a button component that you can re-use and is able to give the user meaningful information whilst being nicely animated; We will then use this button to fetch Pokemon using the [poke API](https://pokeapi.co).

### This is the end result:

[Code + Preview](https://codesandbox.io/s/button-state-finished-1njld?file=/src/App.svelte)

[Preview](https://1njld.sse.codesandbox.io/)

## Before we start 👨‍💻

I'm going to be assuming you've already set up your environment with **Svelte** and **Tailwind CSS**, if you haven't you can follow this [guide](https://dev.to/swyx/how-to-set-up-svelte-with-tailwind-css-4fg5) by [swyx](https://dev.to/swyx) to help you out.

## Let's start 🎉

We're going to create a Button component in `src/components/ui/buttons/Button.svelte` or whatever directory you prefer.

Now let's import the button component where we want it to be displayed - example [here](https://svelte.dev/repl/df2e8d20bc174bfdb1aca436ab967415?version=3.43.1)

```javascript
<script>
	import Button from '../components/ui/buttons/Button.svelte';
</script>

<Button>My button</Button>
```

Let's now set up the states for our button in our Button.svelte and our index page, the four states we are going to have are the default, loading, error and done state.

## index.svelte (or wherever your button is displayed)

Our index file is where we render the Button component, here we're going to handle the click event and control its appearance - to do this we use component `props`. They look like custom HTML attributes and we use them to send data from the parent `index.svelte` to the child `Button.svelte`

Let's now add all of our possible button states and initialise them as `false`. Initialising variables is always recommended as it gives you an idea of what kind of value they hold, in this case they're `booleans`

```javascript
<script>
	// button states
	let isLoading = false;
	let isError = false;
	let isDone = false;
</script>
<Button
	loading={isLoading}
	error={isError}
	done={isDone}
>
	Catch Pokemon
</Button>
```

And let's create three props to control its appearance

```javascript
<Button
	loading={isLoading}
	error={isError}
	done={isDone}
	loadingClass="bg-yellow-600 scale-110 active:bg-yellow-600"
	errorClass="bg-red-600 scale-110 shake active:bg-red-600"
	doneClass="bg-green-600 scale-110 active:bg-green-600"
>
	Catch Pokemon
</Button>
```

Don't worry too much about the `shake` class for now, we're going to create the CSS for it later.

If you're not familiar with it, all of these classes except for `shake` are `tailwindcss` classes - more info on them [here](https://tailwindcss.com).

## Button.svelte

In our Button component we're then going to use svelte's own `export let yourVariable` to read what the parent component is sending to us - note they're also initialised with a value so when our component is mounted we know what they are, they can be manipulated from `index.svelte`.

Initialising these class variables with an empty string `''` prevents rendering class names of `undefined` in our HTML on mount.

Let's now add these state initialisation variables and a default base class for our button:

```javascript
<script>
	// class variables
	export let loadingClass = '';
	export let errorClass = '';
	export let doneClass = '';
	// state variables
	export let loading = false;
	export let error = false;
	export let done = false;
</script>

<button
	class="transition-all overflow-hidden transform relative text-white px-4 py-3 rounded-lg shadow-lg"
>
	<slot/>
</button>
```

Now using ternary operators we can conditionally set a class based on which of the three states we're in

If you're not familiar with ternary operators here's how they work:

```javascript
{
	loading ? loadingClass : '';
}
```

this means **if** loading is `true` use `loadingClass` **else** use an empty string `''`

Let's add these in! 👏

```javascript
<button
	class="transition-all overflow-hidden transform relative text-white px-4 py-3 rounded-lg shadow-lg {loading
		? loadingClass
		: ''} {error ? errorClass : ''} {done ? doneClass : ''} {loading || error || done
		? 'pr-8 pl-4'
		: 'bg-blue-400 hover:bg-blue-600'}
  "
	on:click|preventDefault
>
	<slot />
</button>
```

Notice I've added an `on:click|preventDefault` attribute on it, this means we can now use `on:click` events directly on our Button component in `index.svelte`

`{loading || error || done ? 'pr-8 pl-4' : 'bg-blue-400 hover:bg-blue-600'}`

This line sets the default background + hover and changes the padding if any of the states is true (the right padding change will be needed for our icon)

## Let's add our icons to Button.svelte!

Source: [Heroicons](https://heroicons.com)

I've picked three icons from the web for this - Don't exactly remember the sources for all of them so please let me know in the comments if you know who made these!

We're going to want these icons to be animated and to appear/disappear based on our loading/error/done states so let's add our code with transitions right after our `slot`

Let's import `fly` from svelte transitions and `quintInOut` from svelte easing to animate them

```javascript
import { fly } from 'svelte/transition';
import { quintInOut } from 'svelte/easing';
```

and let's create a default class for all the icons to position them correctly

```javascript
<script>
	import {fly} from 'svelte/transition'; import {quintInOut} from 'svelte/easing'; // class
	variables export let loadingClass = ''; export let errorClass = ''; export let doneClass = ''; //
	state variables export let loading = false; export let error = false; export let done = false; let
	iconClass = 'absolute right-2 top-2/4 transform -translate-y-2/4 ';
</script>
```

Our icon will have position absolute, relative to its button parent and vertically aligned in the middle thanks to the utility classes `top-2/4 transform -translate-y-2/4`

## Creating our icons!

Let's now add our icons to our `Button.svelte` component right after our `slot` tag

We are going to need an If block for our different states

```javascript
{#if loading}
  <span class={iconClass}>
    loading icon here
  </span>
{:else if error}
  <span class={iconClass}>
    error icon here
  </span>
{:else if done}
  <span class={iconClass}>
    done icon here
  </span>
{/if}
```

We're wrapping them in a `span` tag so we can use a svelte transition attribute on them.

This is the code for all the icons with the styles:

```javascript

<button
	class="transition-all overflow-hidden transform relative text-white px-4 py-3 rounded-lg shadow-lg {loading
		? loadingClass
		: ''} {error ? errorClass : ''} {done ? doneClass : ''} {loading || error || done
		? 'pr-8 pl-4'
		: 'bg-blue-400 hover:bg-blue-600'}
  "
	on:click|preventDefault
>
	<slot />

	{#if loading}
		<span
			in:fly|local={{ duration: 600, y: 30, easing: quintInOut }}
			out:fly|local={{ duration: 300, y: 30 }}
			class={iconClass}
		>
			<svg class="spinner" viewBox="0 0 50 50">
				<circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5" />
			</svg>
		</span>
	{:else if error}
		<span
			in:fly|local={{ duration: 600, x: 30, easing: quintInOut }}
			out:fly|local={{ duration: 300, x: 30 }}
			class={iconClass}
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-5 w-5 fill-current"
				viewBox="0 0 20 20"
				fill="currentColor"
			>
				<path
					fill-rule="evenodd"
					d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z"
					clip-rule="evenodd"
				/>
			</svg>
		</span>
	{:else if done}
		<span
			in:fly|local={{ duration: 600, x: 30, easing: quintInOut }}
			out:fly|local={{ duration: 300, x: 30 }}
			class={iconClass}
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-5 w-5"
				viewBox="0 0 20 20"
				fill="currentColor"
			>
				<path
					fill-rule="evenodd"
					d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
					clip-rule="evenodd"
				/>
			</svg>
		</span>
	{/if}
</button>

<style>
	.spinner {
		animation: rotate 2s linear infinite;
		z-index: 2;
		width: 20px;
		height: 20px;
		z-index: 15;
	}
	.path {
		stroke: white;
		stroke-linecap: round;
		animation: dash 1.5s ease-in-out infinite;
	}
	@keyframes rotate {
		100% {
			transform: rotate(360deg);
		}
	}
	@keyframes dash {
		0% {
			stroke-dasharray: 1, 150;
			stroke-dashoffset: 0;
		}
		50% {
			stroke-dasharray: 90, 150;
			stroke-dashoffset: -35;
		}
		100% {
			stroke-dasharray: 90, 150;
			stroke-dashoffset: -124;
		}
	}
	.shake {
		animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97);
		transform: translate3d(0, 0, 0);
		backface-visibility: hidden;
		perspective: 1000px;
	}
	@keyframes shake {
		10%,
		90% {
			transform: translate3d(-2px, 0, 0);
		}

		20%,
		80% {
			transform: translate3d(4px, 0, 0);
		}

		30%,
		50%,
		70% {
			transform: translate3d(-6px, 0, 0);
		}

		40%,
		60% {
			transform: translate3d(6px, 0, 0);
		}
	}
</style>
```

We're using different duration values for in and out because we want the animation to leave quicker than it comes in to leave room for the next icon to take the spotlight.

the `shake` and `spinner` classes are for the error animation and the spinner respectively, you can use any other icon here, this is just as an example.

## NICE 🥳

Our button component is now finished and it should look like this:

```javascript
<script>
	import { fly } from 'svelte/transition';
	import { quintInOut } from 'svelte/easing';
	// class variables
	export let loadingClass = '';
	export let errorClass = '';
	export let doneClass = '';
	// state variables
	export let loading = false;
	export let error = false;
	export let done = false;

	let iconClass = 'absolute right-2 top-2/4	transform -translate-y-2/4 ';
</script>

<button
	class="transition-all overflow-hidden transform relative text-white px-4 py-3 rounded-lg shadow-lg {loading
		? loadingClass
		: ''} {error ? errorClass : ''} {done ? doneClass : ''} {loading || error || done
		? 'pr-8 pl-4'
		: 'bg-blue-400 hover:bg-blue-600'}
  "
	on:click|preventDefault
>
	<slot />

	{#if loading}
		<span
			in:fly|local={{ duration: 600, y: 30, easing: quintInOut }}
			out:fly|local={{ duration: 300, y: 30 }}
			class={iconClass}
		>
			<svg class="spinner" viewBox="0 0 50 50">
				<circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5" />
			</svg>
		</span>
	{:else if error}
		<span
			in:fly|local={{ duration: 600, x: 30, easing: quintInOut }}
			out:fly|local={{ duration: 300, x: 30 }}
			class={iconClass}
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-5 w-5 fill-current"
				viewBox="0 0 20 20"
				fill="currentColor"
			>
				<path
					fill-rule="evenodd"
					d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z"
					clip-rule="evenodd"
				/>
			</svg>
		</span>
	{:else if done}
		<span
			in:fly|local={{ duration: 600, x: 30, easing: quintInOut }}
			out:fly|local={{ duration: 300, x: 30 }}
			class={iconClass}
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-5 w-5"
				viewBox="0 0 20 20"
				fill="currentColor"
			>
				<path
					fill-rule="evenodd"
					d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
					clip-rule="evenodd"
				/>
			</svg>
		</span>
	{/if}
</button>

<style>
	.spinner {
		animation: rotate 2s linear infinite;
		z-index: 2;
		width: 20px;
		height: 20px;
		z-index: 15;
	}
	.path {
		stroke: white;
		stroke-linecap: round;
		animation: dash 1.5s ease-in-out infinite;
	}
	@keyframes rotate {
		100% {
			transform: rotate(360deg);
		}
	}
	@keyframes dash {
		0% {
			stroke-dasharray: 1, 150;
			stroke-dashoffset: 0;
		}
		50% {
			stroke-dasharray: 90, 150;
			stroke-dashoffset: -35;
		}
		100% {
			stroke-dasharray: 90, 150;
			stroke-dashoffset: -124;
		}
	}
	.shake {
		animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97);
		transform: translate3d(0, 0, 0);
		backface-visibility: hidden;
		perspective: 1000px;
	}
	@keyframes shake {
		10%,
		90% {
			transform: translate3d(-2px, 0, 0);
		}

		20%,
		80% {
			transform: translate3d(4px, 0, 0);
		}

		30%,
		50%,
		70% {
			transform: translate3d(-6px, 0, 0);
		}

		40%,
		60% {
			transform: translate3d(6px, 0, 0);
		}
	}
</style>
```

it's now time to go back to our `index.svelte` file to fetch our Pokemons!

## Use pokeAPI to get our Pokemons 🐛

Consuming APIs in Svelte is **really** easy, we are going to use the fetch API and svelte's `await` blocks to do the job.

your index file should look something like this at this point

```javascript
<script>
	import Button from '../components/ui/buttons/Button.svelte';

	// button states
	let isLoading = false;
	let isError = false;
	let isDone = false;
</script>

<div class="flex my-8 justify-center">
	<Button
		loading={isLoading}
		error={isError}
		done={isDone}
		loadingClass="bg-yellow-600 scale-110 active:bg-yellow-600"
		errorClass="bg-red-600 scale-110 shake active:bg-red-600"
		doneClass="bg-green-600 scale-110 active:bg-green-600"
	>
  Catch Pokemon
	</Button>
</div>

```

First of all, let's add some initial state to hold our pokemons

```javascript
let pokemons = null;
```

this `pokemons` variable will be populated with the response from our API call, let's now browse to the [pokeAPI](<(https://pokeapi.co){:target="_blank"}>) website to check how to query for what we want.

For the purpose of this tutorial we will only fetch 20 pokemons but you'll be able to adjust the limit to your liking.

Here's our endpoint with our query
`https://pokeapi.co/api/v2/pokemon?limit=20`

let's create a variable for the limit and let's also add a `delay` one we're going to use for our state changes

```javascript
let pokemons = null;
let limit = 20;
let delay = 2000;
```

now that our variables are set we can proceed with creating a function that will fetch our data, since fetch returns a promise, we can use `async/await` to get our pokemons

```javascript
async function fetchPokemon() {
	const response = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=${limit}`);
	return response.json();
}
```

note that we're using our `limit` variable to set how many pokemons we want to fetch; Meaning we're probably never going to have to touch this function again.

Nothing is happening yet, we still have to call our function, let's call it when we click on our `Button` component and let's also move it to the centre of the page.

To move it to the centre we'll just wrap it in a flex div and justify its content to the centre, like so

```javascript
<div class="flex my-8 justify-center">
	<Button
		loading="{isLoading}"
		error="{isError}"
		done="{isDone}"
		loadingClass="bg-yellow-600 scale-110 active:bg-yellow-600"
		errorClass="bg-red-600 scale-110 shake active:bg-red-600"
		doneClass="bg-green-600 scale-110 active:bg-green-600"
	>
		Catch Pokemon
	</Button>
</div>
```

and add an on click event to it, with a function that we still have to write called `handleButton`

```javascript
<Button
  on:click={handleButton}
  ...
```

before we write the function we can add different text based on the state, like so:

```javascript
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
```

## the handleButton function

this function is what is going to control what happens when you press the `Button` component, I'm going to use `setTimeouts` to artificially delay the loading state, this is because our 20 pokemon request is usually super quick and you wouldn't be able to see the state at all othwerwise - ideally the loading state should change to "done" right after the response comes from the API.

Let's write the function

```javascript
function handleButton() {
	// we only fetch once on this demo
	// this is so we can display "error"
	// if someone tries to fetch twice
	if (!pokemons) {
		// this sets our pokemons variable
		// to the API response
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
```

Another way of doing this without the artificial delay would be adding the loading state to the fetchPokemon function and using a reactive state, **just as a quick example**:

```javascript
$: if (pokemons?.length) {
	isLoading = false;
}

async function fetchPokemon() {
	isLoading = true;
	const response = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=${limit}`);
	return response.json();
}
```

## Display our pokemons

There are several ways to do this but a nice and simple way is to use svelte's `await` blocks

```javascript
{#await promise}
  Loading message...
{:then result}
  Use your {result}
{:catch error}
  Handle the error {error}
{/await}
```

I'm going to be use grid to display the pokemons and a fade transition for the safety check, first let's check if the `pokemons` variable is populated

```javascript
{#if pokemons}
  <div
		transition:fade={{ duration: 800, easing: quintInOut }}
		class="grid grid-cols-2 lg:grid-cols-5 gap-8 my-8"
	>
  {#await pokemons}
    Loading...
  {:then result}
    Use your {result}
  {:catch error}
    Handle the error {error}
  {/await}
  </div>
{/if}
```

the fade transition will happen when the condition is met so when `pokemons` is not a `falsy` value

All we need to do now is to create an each loop using svelte's `each` blocks to loop through our results and render each individual pokemon, let's get inside `{:then result}`

```javascript
  {#await pokemons}
			Loading...
	{:then result}
		{#each result.results as pokemon, i}
      <div
        class="border border-gray-600 p-8 rounded-xl text-white bg-gray-800 hover:bg-gray-900 shadow-lg capitalize"
        transition:fly={{ duration: 200, y: 30, delay: i * 100 }}
      >
        <h3 class="text-2xl font-extrabold">{pokemon.name}</h3>
        <h5 class="text-base">Pokemon #{i + 1}</h5>
      </div>
    {/each}
	{:catch error}
		An error has occurred {error}
	{/await}
```

let's break this down:

`result` will be our response object, as you can see from [here](https://pokeapi.co/api/v2/pokemon?limit=20)

what we want from this object is the key `results` which holds all our 20 pokemons, so this is how we loop through them:

```javascript
{#each result.results as pokemon, i}
  individual pokemon here {pokemon.name}
{#each}
```

`i` would be our index, but we can also use this to identify the pokemon number, which will be useful to grab the relative image for each one of them, we just need to make a simple change.

Indexes start at `0` in javascript but our first pokemon would be `1`, all we need to do is add `1` to our index to find out our Pokemon number.

to fetch the images I've had a look at a sample pokemon response from [here](https://pokeapi.co/api/v2/pokemon/ditto) and found that the image URLs follow this pattern:

`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/{pokemonNumber}.png`

where `pokemonNumber` woud be our `i + 1` in our loop.

All together should look like this:

```javascript
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
```

## ALL DONE! 👏👏

Our very simple button state application is now finished, the end result is [here](https://codesandbox.io/s/button-state-finished-1njld?file=/src/App.svelte) if you need any help referencing the code.

Hope this guide will help you get more familiar with Svelte, thanks for checking it out and let me know in the comments if you have any ideas to improve it further!
