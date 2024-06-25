<script>
    import { push } from "svelte-spa-router";
    import { onMount } from "svelte";
    import axios from "axios";

    let star = null;
    let error = null;
    let id = null;

    onMount(async () => {
        const hashFragment = window.location.hash.substring(1);
        id = hashFragment.split("/").pop();

        try {
            const response = await axios.get("/data/stardata.json");
            const jsonData = response.data;
            star = jsonData.find((s) => s.id === parseInt(id, 10));

            if (!star) {
                throw new Error("Star not found");
            }
        } catch (err) {
            console.error("Error fetching data:", err);
            error = err.message;
        }
    });

    function goBack() {
        push("/");
    }
</script>

<main>
    <button on:click={goBack}>Back to Home</button>
    {#if error}
        <p>Error: {error}</p>
    {:else if star}
        <div>
            <h1>{star.proper}</h1>
            <p><strong>Magnitude:</strong> {star.mag}</p>
            <p>
                <strong>Coordinates:</strong> (x: {star.x0}, y: {star.y0}, z: {star.z0})
            </p>
            {#if star.wikiUrl}
                <p><a href={star.wikiUrl} target="_blank">Wikipedia</a></p>
            {/if}
        </div>
    {:else}
        <p>Loading...</p>
    {/if}
</main>
