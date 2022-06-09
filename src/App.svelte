<script>
  import { Router, Route, Link } from 'svelte-navigator'
  import Auction from './Auction.svelte';
  import axios from 'axios'

  let auctions = getAuctions()

  async function getAuctions() {
        const response = await axios.get(`http://localhost:8000/auctions`)
        auctions = response.data
        return await response.data
    }

</script>

<main>
  <Router>
    <Route path="/">
      <h1> Auctions </h1>
      {#await auctions}
        <p></p>
      {:then auctions} 
      <div class="grid"> 
        {#each auctions as au}
          <div class="card"> 
          <Link to="auction/{au.id}">
            <p class="info">Go to auction {au.id} </p>
            
          </Link>
          <p class="info">{au.completed ? "Is finished" : 'Will start soon'} </p>
        </div>
        {/each}
      </div>
      {/await}
      
      
    </Route>

    <Route path="/auction/:id" component={Auction}>
    </Route>
  </Router>
</main>

<style>
    .card{
      padding: 5px;
      border: 1px solid grey;
    }
    .grid {
        display: grid;
        gap: 10px;
        grid-template-columns: auto auto auto auto;
    }
    .info {
      margin-left: 2em;
    }
    h1 {
      text-align: center;
    }
</style>
