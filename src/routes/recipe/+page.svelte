<script>
	import '../../app.css';
	import {
		Button,
		Card,
		Listgroup
	} from 'flowbite-svelte';
	import { onMount } from 'svelte';

	let width;

  let id;
	let recipe = null;
	
	onMount(loadRecipe);

	//bindet erhaltene Daten von getRecipe()in unser system ein
	function loadRecipe() {
	  const queryString = window.location.search;
	  const urlParams = new URLSearchParams(queryString);
	  id = urlParams.get("id");
		console.log("load recpie with id '" + id + "'...");
		getRecipe().then(new_recipe => {
			console.log("recieved recipe:");
			console.log(new_recipe);
			recipe = new_recipe;
		});
	}

	//Api Informationen
	import SECRET_API_KEY from '../../key.json';
	const url = 'https://tasty.p.rapidapi.com/recipes/get-more-info';
	const options = {
		method: 'GET',
		headers: {
			'X-RapidAPI-Key': '',
			'X-RapidAPI-Host': 'tasty.p.rapidapi.com'
		}
	};

	//API anfrage raus 
	async function getRecipe() {
		console.log('request recipe...');
		options.headers['X-RapidAPI-Key'] = SECRET_API_KEY.secret;
		let fullUrl = url + "?id=" + id.toString();
		console.log("full url: " + fullUrl);
		const response = await fetch(fullUrl, options);
		const text = await response.text();
		console.log(JSON.parse(text));
		return JSON.parse(text);
		/*
		const response = await fetch("/src/rezeptdummy.json");
		const text = await response.text();
		return JSON.parse(text).results[0];
		*/
	}
	
	//wandelt Datum in Text um 
	function formatDate(date) {
	  return date.getDay() + "." + date.getMonth() + "." + date.getFullYear();
	}
</script>

<svelte:window bind:innerWidth={width} />

<div id="flex-container">
	<div id="heading">
	  {#if recipe != null}
	    <p
		    style="color:white; font-size: 30px; margin: 40px 0px 20px 0px; text-align: center"
	    >
		    Rezept
	    </p>
		{/if}
	</div>
	<div style="margin: 10px 0 0 10px">
		<Button outline color="purple" size="sm" href="/">Zurück</Button>
  </div>
	<div id="kacheln">
		{#if recipe != null}
			<Card img={recipe.thumbnail_url}>
		    <p
			    style="font-size: 30px; margin: 10px 0px 15px 0px"
		    >
			    {recipe.name}
		    </p>
				<p>{recipe.description}</p>
			</Card>
			<div>
			  {#each recipe.topics as tag}
			    <Button class="focus:ring-white" pill outline color="purple" style="margin: 18px 10px 0px 0px">{tag.name}</Button>
		    {/each}
			</div>
			<Card style="margin-top: 15px">
				<p>
				  {#if recipe.total_time_minutes != null && recipe.total_time_minutes != 0}
				    Dauer: {recipe.total_time_minutes} min<br>
				  {/if}
				  {#if recipe.nutrition.calories != null && recipe.nutrition.calories != 0}
				    {recipe.nutrition.calories} Kalorien<br>
				  {/if}
				  Erstellt am {formatDate(new Date(recipe.created_at * 1000))} 
				</p>
			</Card>
			<Card style="margin-top: 15px">
		    <p
			    style="font-size: 20px; margin: 0px 0px 10px 0px"
		    >
			    Zutaten
		    </p>
			  {#each recipe.sections[0].components as item}
			    {#if item.raw_text != "n/a"}
            <p>{item.raw_text}</p>
			    {:else}
            <p>{item.ingredient.name}</p>
			    {/if}
        {/each}
			</Card>
			<Card style="margin: 15px 0px 20px 0px">
		    <p
			    style="font-size: 20px; margin: 0px 0px 10px 0px"
		    >
			    Anleitung
		    </p>
        <Listgroup class="border-0" items={recipe.instructions} let:item>
          <p>{item.display_text}</p>
        </Listgroup>
			</Card>
		{/if}
	</div>
</div>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Titillium+Web:wght@200;300;400;600;700;900&display=swap');
	
	p {
	  font-family: "Titillium Web"
	}
	
	#kacheln {
		margin: 20px 10px 0px 10px;
	}
	
	#flex-container {
		display: flex;
		flex-direction: column;
	}
	
	#heading {
		background-color: #242b32;
		z-index: 0;
	}
</style>
