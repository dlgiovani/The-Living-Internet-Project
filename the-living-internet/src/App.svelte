<script lang="ts">

  let database = [
	{
          url: "https://dlgiovani.github.io",
	  path_to_li: "/",
	  status: null,
	  description: null,
	  thumbnail_url: null
	},
  ]

  let favourites = localStorage.getItem("favourites") | null;
  if (favourites) {
    database = [...favourites, ...database];
  } // append favourites to the beggining of the db and it really does not matter that they'll be fetched twice
  
  document.addEventListener('DOMContentLoaded', async () => {
    for (let i = 0; i < database.length; i++) {
     // TODO: make validation to also cause fetch if the data is too old or if it is a favourite.
      if ((!database[i].status) || database[i].status >= 400) {
        const result = await fetch(database[i].url + database[i].path_to_li + '/living_internet/index.json');
	const data = await result.json();
	database[i].status = result.status_code;
	database[i].description = data.description;
	database[i].thumbnail_url = database[i].url + 
				    database[i].path_to_li + 
				    "/living_internet" + data.thumbnail_url;
      }
    }

  });

  let search = '';
  let results = [];
  async function handleSearch() {
    console.log(search);
    results = [];
    const batchSize = 10;
    for (let i = 0; i < database.length; i += batchSize) {
      const batch = database.slice(i, i + batchSize).map((o) => ({...o}));
      const batchResults = batch.filter((item) => 
      			item.url.includes(search) || item.description.includes(search));
      results = [...results, ...batchResults];
    }
  }

</script>

<main>
  <input type="text" bind:value={search}/>
  <button type="submit" on:click={async () => handleSearch()}>search!</button>

  {#each results as result}
    <a target="_blank" href={result.url} class="result">
      <div>
        <img src={result.thumbnail_url} alt={result}/>
	<div>
	  <h2>{result.url}</h2>
          <p>{result.description}</p>
	</div>
      </div>
    </a>
  {/each}
</main>

<style>
  input {
    height: 2rem;
    width: 800px;
    max-width: 80%;
    margin-bottom: 5vh;
  }

  .result {
    display: block;
    & div {
      display: flex;
      flex-direction: row;
      gap: 10px;

      & img {
        height: 100%;
      }

      & div {
        display: flex;
	flex-direction: column;
	align-items: start;
	
	& h2 {
	  padding: 0;
	  margin: 0;
	}
      }
    }
  }
</style>
