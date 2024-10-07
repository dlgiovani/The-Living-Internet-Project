<script lang="ts">

let database = [
  {
    resource_url: "https://dlgiovani.github.io/living_internet/index.json",
    last_successful_fetch: null,
    last_fetch: null,
    content: null
  },
]

let favourites = localStorage.getItem("favourites") | null;
if (favourites) {
  database = [...favourites, ...database];
} // append favourites to the beggining of the db and it really does not matter that they'll be fetched twice

const fetchData = async () => {
  for (let i = 0; i < database.length; i++) {
    console.log(database[i].last_successful_fetch);
    if (Date.parse(database[i].last_successful_fetch < (Date.now() - 1000 * 60 * 10)) || !database[i].last_successful_fetch ) {
      const result = await fetch(database[i].resource_url);
      database[i].last_fetch = Date.now()
      if (result.status <= 200) {
        database[i].last_successful_fetch = Date.now()
       
        database[i].content = await result.json();
      }
    }
  }
  console.log(database);
}

fetchData();

let search = '';
let siteResults = [];
let postResults = [];
async function handleSearch() {
  siteResults = [];
  postResults = [];
  let upperSearch = search.toUpperCase();
  const batchSize = 10;
  for (let i = 0; i < database.length; i += batchSize) {
    const batch = database.slice(i, i + batchSize).map((o) => ({...o}));

    const batchResults = batch.filter((item) => {
      if (item.content.url.toUpperCase().includes(upperSearch) || item.content.title.toUpperCase().includes(upperSearch)) {
        const { feed, ...content } = item.content;
        siteResults = [...siteResults, content]
      }
    }
    );

    const batchContentResults = batch.map((item) => 
      item.content.feed.filter((post) =>  {    
        if (JSON.stringify(post).toUpperCase().includes(upperSearch)) {

          postResults = [...postResults, { ...post, origin: item.content.title + ' | ' + item.content.url}]
        }
      }));
  }

  console.log(JSON.stringify(siteResults));
}

</script>

<main>
  <input type="text" bind:value={search}/>
  <button type="submit" on:click={async () => handleSearch()}>search!</button>

  {#each siteResults as result}
    <a target="_blank" href={result.url} class="result">
      <h2>{result?.title}</h2>
      <p>{result.url}</p>
    </a>
  {/each}

  {#each postResults as pResult}
    <a target="_blank" href={pResult?.url} class="result">
      <div>
        {#if (pResult?.images) }
          <img width=200 src={pResult?.images[0]?.url} alt={pResult?.images[0]?.alt}/>
        {/if}
        <div>
          <p>From: {pResult.origin}</p>
          <h2>{pResult.title}</h2>
          <p>{pResult.url}</p>
          <p>{pResult.description}</p>
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
  border: solid 1px white;
  margin: 10px 0;
  padding: 6px;
  &:hover {
    background-color: #ffffff11;
  }
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
