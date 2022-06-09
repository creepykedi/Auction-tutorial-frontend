<script>
    import axios from 'axios'
    import {lastMsgs} from './Store'
    import { useParams } from "svelte-navigator";

    const params = useParams();
    

    $: item = getItem()
    let id = $params.id
    $: currentBid = 0
    let isFinished = false
    const participant = Math.floor(Math.random() * 100) + 1
    let auctionEnds 
    $: newBid = getMinBid(item, currentBid)
    $: countdown = (auctionEnds - new Date() ) / 1000
    let interval

    const getMinBid = (item, currentBid) => {
        return item.then(i => newBid = currentBid ? Math.max(i.item.min_price, currentBid) + i.item.price_step :
            Math.max(i.item.min_price, currentBid)
        ) 
    }

    async function getItem() {
        const response = await axios.get(`http://localhost:8000/auction/${id}`)
        currentBid = response.data.item.bid
        return await response.data
    }

    function isJsonString(str) {
        try {
            JSON.parse(str)
        } catch (e) {
            return false
        }
        return true
    }

    const ws = new WebSocket(`ws://localhost:8000/auction/${id}/ws/${participant}`)

    ws.onmessage = (event) => {
        let incoming = event.data
        if (isJsonString(incoming)) {
            incoming = JSON.parse(incoming)
            if (incoming.new_price) {
                currentBid = incoming.new_price
                auctionEnds = new Date(incoming.ends)
                clearInterval(interval)
                interval = setInterval(() => {
                    countDown()
                }, 1000)
            }
            else if (incoming.json_data.completed) {
                isFinished = incoming.json_data.completed
            }
        }
        else {
            $lastMsgs = [...$lastMsgs, incoming]
        }
    }

    const makeNewBid = (e) => {
        const data = JSON.stringify({bid: newBid})
        ws.send(data)
        e.preventDefault()
    }

    const countDown = () => {
        if (countdown > 0) {
            countdown -= 1
        }   
        else {
            clearInterval(interval)
            countdown = 0
            isFinished = true
        }
    }

</script>

<div class="grid">
    <div class="container">
        {#if !isFinished}
            <p> Today's item is...</p>
        {/if}
        {#await item}
            <p> Loading info...</p>
        {:then item} 
        <img src="https://media.npr.org/assets/img/2012/05/03/scream_vert-0af6ef30e3859e9b8c92ca5070ba9f8f991a5ce9.jpg"
        alt='painting'>
            <p>{item.item.item_name}</p>
            <p class="description">{item.item.item_description}</p>

            {#if !currentBid}
                <p>The starting price is ${item.item.min_price}</p>
            {:else}
                <p>{#if !isFinished} Current bid is {:else} The last bid was {/if} ${currentBid} </p> 
            {/if}
            <button disabled={isFinished} on:click={makeNewBid}> Bid $ {newBid} </button>
        {/await}
        </div>

        <div>
            <p>Welcome, participant # {participant}!</p>
            {#each $lastMsgs as msg}
                <h4> {msg} </h4>
            {/each}
        </div>
        {#if !isNaN(auctionEnds)}
            {#if countdown > 0}
                <p>Auction is ending in {countdown.toFixed(1)}!</p>
                {:else}
                <p>Auction ended!</p>
            {/if}
        {/if}
    
</div>

<style>
    .container {
        display: flex;
        justify-self: center;
        flex-direction: column;
        max-height: 100%;
    }
    .grid {
        display: grid;
        gap: 10px;
        grid-template-columns: auto auto;
    }
    img {
        max-width: 100%;
        height: auto;
        border: solid 2px black;
    }
    .description {
        font-style: italic;
    }
</style>