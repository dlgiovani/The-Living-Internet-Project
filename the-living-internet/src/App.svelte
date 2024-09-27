<script lang="ts">
  let search = '';
  let database = [
	"https://dlgiovani.github.io",
  ]
  let results = [];

  function handleSearch() {
    // compare search with database of indexed websites
    // present the results
    Promise.all(database.filter((item) => item.includes(search)).map(async(result) => {
      let request = await fetch(`${result}/living_internet/index.json`);
      let data = await request.json();
      results = [...results, { url: `${result}${data.root}`, thumb: `${result}/living_internet${data.thumbnail_url}`, description: data.description }]
    }));
  }

</script>

<main>
  <input type="text" bind:value={search}/>
  <button type="submit" on:click={handleSearch}>search!</button>

  {#each results as result}
    <a target="_blank" href={result.url} class="result">
      <div>
        <img src={result.thumb} alt={result}/>
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
