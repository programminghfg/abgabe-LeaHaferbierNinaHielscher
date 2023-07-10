<script>
	import '../app.css';
	import { browser } from '$app/environment'; // Wird es im Browser oder im Server ausgeführt? (Weil window gibt es nur im Browser)
	import {
		Sidebar,
		SidebarGroup,
		SidebarItem,
		SidebarWrapper,
		Drawer,
		CloseButton,
		Button,
		Card,
		Search
	} from 'flowbite-svelte'; //eine Framework für unser Design 
	import { sineIn } from 'svelte/easing';

	
	// Timer für dumme Suchleiste (eigentlich völlig unnötig), damit die Suchleiste über das Sortieren und Filtern Fenster geh, sobald dies geschlossen wird.
	$: {
		if (drawerHidden && browser) {
			window.setTimeout(() => {
				document.getElementById('suche').style.zIndex = '0';
			}, 200);
		}
	}

	let transitionParams = {
		x: 320,
		duration: 200,
		easing: sineIn
	};

	let width;
	let backdrop = false;
	let activateClickOutside = true;
	let drawerHidden = true;
	let searchBarText = '';

	const sortingButtonNames = ['Dauer (Aufsteigend)', 'Dauer (Absteigend)', 'Neueste', 'Älteste', 'Kalorien (Aufsteigend)', 'Kalorien (Absteigend)'];
	let selectedSorting = '';
	let filteringButtonNames = ['Desserts', 'Vegan', 'Breakfast', 'Lunch', 'Dinner'];
	let selectedFilters = [];

	let tags = [];

	let recipes = [];
	let recipesUntouched = []; // Unsortiert und ungefiltert

	
	// Sortier & Filtern Fenster über Suchleiste
	function showDrawer() {
		drawerHidden = false;
		document.getElementById('suche').style.zIndex = '-1';
	}

	//Wir gehen über alle tags drüber und filtern den raus, der angeklickt wurde.
	function removeTag(tagToRemove) {
		tags = tags.filter((tag) => tag != tagToRemove);
		loadRecipes();
	}

	// Suchitems lassen sich einzeln oder durch Komma getrennt in Tags umwandeln.
	function search(event) {
		if (event.key == 'Enter') {
		  let newTags = searchBarText.split(",");
		  newTags.forEach(newTag => tags.push(newTag));
			tags = tags;
			searchBarText = '';
			loadRecipes();
		}
	}

	//wenn ein Button selected ist mit css stylen weil framework nicht richtig funktioniert
	function updateButtons() {
		let buttons = document.getElementsByName('sortingButton');
		buttons.forEach((element) => {
			if (element.innerHTML.trim() == selectedSorting) {   //warum auch immer hat das was im Button steht ein Leerzeichen! Deshalb nie gleich wie sorting Button name oben im array -> trim()
				element.style.color = "purple";
				element.style.borderColor = "purple";
			} else {
				element.style.color = "grey";
				element.style.borderColor = "lightGrey";
			}
		});

		buttons = document.getElementsByName('filteringButton');
		buttons.forEach((element) => {
			if (arrayContains(selectedFilters, element.innerHTML.trim())) {
				element.style.color = "purple";
				element.style.borderColor = "purple";
			} else {
				element.style.color = "grey";
				element.style.borderColor = "lightGrey";
			}
		});
	}

	//sobal ein Tag hinzugefügt wird, die rezepte neu laden
	function loadRecipes() {
		console.log("load recpies with [" + tags + "]...");
		getRecipes().then(newRecipes => {
			console.log("recieved recipes:");
			console.log(newRecipes);
			/*
			newRecipes.forEach(recipe => {
			  if (recipe.name.length > 25) {
			    console.log(recipe.name);
			    recipe.name = recipe.name.split(0, 20)[0];
			    console.log(recipe.name);
          recipe.name += "...";
			  }
			});
			*/
			recipes = newRecipes;
			recipesUntouched = structuredClone(newRecipes); //von allen Rezepten einen Klon machen (für gefilterte)
			let prevSelectedFilters = selectedFilters;
			updateSortingAndFiltering();
		});
	}

	//Original Version wird wieder geladen, Klon wird erstellt und es wird gefiltert und sortiert 
	function updateSortingAndFiltering() {
		recipes = structuredClone(recipesUntouched);
		filterRecipes();
		sortRecipes();
	}

	//wird ein dort Button angeklickt wird der Array mit den Rezepten mit .sort passend umsortiert.
	function sortRecipes() {
		console.log("apply sorting '" + selectedSorting + "'...");
		switch (selectedSorting) {
			case "":
				break;
			case "Dauer (Aufsteigend)":
				recipes.sort((lhs, rhs) => {
					if (lhs.total_time_minutes > rhs.total_time_minutes) //lhs (lefthandside) rhs(righthandside) = von links nach rechtsaufsteigend
						return 1;
					else if (lhs.total_time_minutes < rhs.total_time_minutes)
						return -1;
					else
						return 0;
				});
				break;
			case "Dauer (Absteigend)":
				recipes.sort((lhs, rhs) => {
					if (lhs.total_time_minutes > rhs.total_time_minutes)
						return -1;
					else if (lhs.total_time_minutes < rhs.total_time_minutes)
						return 1;
					else
						return 0;
				});
				break;
			case "Neueste":
				recipes.sort((lhs, rhs) => {
					if (lhs.created_at > rhs.created_at)
						return -1;
					else if (lhs.created_at < rhs.created_at)
						return 1;
					else
						return 0;
				});
				break;
			case "Älteste":
				recipes.sort((lhs, rhs) => {
					if (lhs.created_at > rhs.created_at)
						return 1;
					else if (lhs.created_at < rhs.created_at)
						return -1;
					else
						return 0;
				});
				break;
			case "Kalorien (Aufsteigend)":
				recipes.sort((lhs, rhs) => {
					if (lhs.nutrition.calories > rhs.nutrition.calories)
						return 1;
					else if (lhs.nutrition.calories < rhs.nutrition.calories)
						return -1;
					else
						return 0;
				});
				break;
			case "Kalorien (Absteigend)":
				recipes.sort((lhs, rhs) => {
					if (lhs.nutrition.calories > rhs.nutrition.calories)
						return -1;
					else if (lhs.nutrition.calories < rhs.nutrition.calories)
						return 1;
					else
						return 0;
				});
				break;
			default:
				console.error("unknown sorting '" + selectedSorting + "'");
				break;
		}
	}
	
	// Hilfsfunktion zum Durchsuchen von Arrays 
	function arrayContains(array, thing) {
	  let contains = false;
	  array.forEach(element => {
	    if (element == thing) {
	      contains = true;
	      return;
	    }
	  });
	  return contains;
	}
	
	function removeFilter(filter) {
	  selectedFilters = selectedFilters.filter(element => {
	    return element != filter
	  });
	}

	//geht über alle Rezepte und filtert die richtigen raus 
	function filterRecipes() {
		console.log("apply filtering '" + selectedFilters + "'...");
		recipes = recipes.filter(recipe => {
			let includeRecipe = true;
			let topicNames = [];
			recipe.topics.forEach(topic => {
			  topicNames.push(topic.name);
			});
			selectedFilters.forEach(filter => {
			  if (!arrayContains(topicNames, filter)) {
			    includeRecipe = false;
			    return;
			  }
			});
			return includeRecipe;
		});
		// alle Bilder mit einem Format das nicht 1:1 ist werden rausgefiltert
		recipes = recipes.filter(recipe => {
			return recipe.aspect_ratio == "1:1"
		});
	}

	//API Informationen
	import SECRET_API_KEY from '../key.json';
	const url = 'https://tasty.p.rapidapi.com/recipes/list';
	const options = {
		method: 'GET',
		headers: {
			'X-RapidAPI-Key': '',
			'X-RapidAPI-Host': 'tasty.p.rapidapi.com'
		}
	};

	//Neue Rezepte von API anfordern 
	async function getRecipes() {
		console.log('request recipes...');
		options.headers['X-RapidAPI-Key'] = SECRET_API_KEY.secret;
		let fullUrl = url + "?from=0&size=20&q=" + tags.toString();
		console.log("full url: " + fullUrl);
		const response = await fetch(fullUrl, options);
		const text = await response.text();
		return JSON.parse(text).results;
		/*
		const response = await fetch("/src/rezeptdummy.json");
		const text = await response.text();
		return JSON.parse(text).results;
		*/
	}
</script>

<svelte:window bind:innerWidth={width} /> 

<!-- das ist das Sortieren und Filtern Fenster -->
<Drawer
	placement="right"
	transitionType="fly"
	{backdrop}
	{transitionParams}
	bind:hidden={drawerHidden}
	bind:activateClickOutside
	width="w-64"
	id="sidebar"
	divClass="bg-white"
>
	<div class="flex items-center">
		<CloseButton on:click={() => (drawerHidden = true)} />
	</div>
	<Sidebar asideClass="w-54">
		<SidebarWrapper class="overflow-y-auto py-4 px-3 rounded bg-white">
			<SidebarGroup>
			  <p>Sortieren</p>
			  {#each sortingButtonNames as sortingButtonName}
				  <Button
					  name="sortingButton"
					  on:click={() => {   
						  if (selectedSorting == sortingButtonName) {
							  selectedSorting = '';
						  } else {
							  selectedSorting = sortingButtonName;
						  }
						  updateButtons();
						  updateSortingAndFiltering();
					  }}
					  outline
					  color="light"
					  class="focus:ring-white w-full"
				  >
					  {sortingButtonName}
				  </Button>
	      {/each}
	      {#if filteringButtonNames.length > 0}
	        <p>Filtern</p>
          {#each filteringButtonNames as filteringButtonName}
	          <Button
		          name="filteringButton"
		          on:click={() => {
			          if (arrayContains(selectedFilters, filteringButtonName)) {
				          removeFilter(filteringButtonName);
			          } else {
				          selectedFilters.push(filteringButtonName);
			          }
			          updateButtons();
			          updateSortingAndFiltering();
		          }}
		          outline
		          color="light"
		          class="focus:ring-white w-full"
	          >
		          {filteringButtonName}
	          </Button>
          {/each}
        {/if}
			</SidebarGroup>
		</SidebarWrapper>
	</Sidebar>
</Drawer>

<div id="flex-container">
	<div id="suche">
		<p
			style="color:white; font-size: 30px; margin: 40px 0px 20px 0px; text-align: center"
		>
			Rezepte
		</p>
		<p
			style="color:white; margin: 0px 0px 10px 20px; font-size:12px; font-weight:200"
		>
			Suche nach weiteren Lebnesmitteln, die du gerne <br /> verwenden würdest:
		</p>
		<div style=" margin:0px 20px 50px 20px;">
			<Search
				on:keypress={(event) => search(event)}
				bind:value={searchBarText}
				size="sm"
				class="rounded-full"
				placeholder="z.B. Banane"
			/>
		</div>
	</div>
	<div id="tags">
		{#each tags as tag}
			<Button
				class="focus:ring-white"
				on:click={() => removeTag(tag)}
				pill
				color="purple"
				style="margin-left: 10px"
				>{tag} <img src="src/img/close.svg" style="display:inline; margin-left: 8px"/></Button
			>
		{/each}
	</div>

	<div id="sortieren">
		<button
			style="margin-right: 10px; float: right;"
			on:click={() => {
				showDrawer();
				window.setTimeout(updateButtons, 200);
			}}
			><img src="src/img/Sort Icon.svg" style="display:inline" /> Sortieren/Filtern
		</button>
	</div>

	<div id="kacheln">
		{#each recipes as recipe}
			<Card href="/recipe?id={recipe.id}" img={recipe.thumbnail_url}>
				<p>
					<!-- Es wird überprüft ob die Kalorien angegeben sind und ob da auch eine Andere zahl als 0 steht und dann werden sie angezeigt-->
				  {recipe.name}<br>
				  {#if recipe.total_time_minutes != null && recipe.total_time_minutes != 0}
				    {recipe.total_time_minutes} min<br> 
				  {/if}
				  {#if recipe.nutrition.calories != null && recipe.nutrition.calories != 0}
				    {recipe.nutrition.calories} Kalorien
				  {/if}
				</p>
			</Card>
		{/each}
	</div>
</div>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Titillium+Web:wght@200;300;400;600;700;900&display=swap');
	
	p {
	  font-family: "Titillium Web"
	}

	#tags {
		overflow: auto;
		white-space: nowrap;
		scrollbar-width: none;
		margin: 20px 0px 20px 0px;
	}
	
	#tags::-webkit-scrollbar {
		display: none;
	}

	#kacheln {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		margin: 20px 10px 20px 10px;
		column-gap: 10px;
		row-gap: 10px;
	}
	
	#flex-container {
		display: flex;
		flex-direction: column;
	}
	
	#suche {
		background-color: #242b32;
		z-index: 0;
	}
</style>
