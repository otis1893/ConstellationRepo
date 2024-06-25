<script>
    import { onMount } from "svelte";
    import { push } from "svelte-spa-router";
    import axios from "axios";

    let data = [];
    let error = null;

    onMount(async () => {
        try {
            const response = await axios.get("/data/stardata.json");
            const jsonData = response.data;
            data = jsonData.filter((star) => star.proper); // Filtere Sterne ohne 'proper'-Name
        } catch (err) {
            console.error("Error fetching data:", err);
            error = err.message;
        }
    });

    function handleButtonClick(id) {
        push(`/detail/${id}`);
    }
</script>

<main>
    {#if error}
        <p>Error: {error}</p>
    {:else if data.length > 0}
        <div>
            <h1>Star Data</h1>
            <p>Check the console when you click a button.</p>
            {#each data.slice(0, 6) as star (star.id)}
                <button
                    on:click={() => handleButtonClick(star.id)}
                    id={"button-" + star.id}
                >
                    {star.proper}
                </button>
            {/each}
        </div>
    {:else}
        <p>Loading...</p>
    {/if}
</main>
