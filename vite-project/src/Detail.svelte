<script>
    import { push } from "svelte-spa-router";
    import { onMount, tick } from "svelte";
    import axios from "axios";

    let star = null;
    let data = [];
    let error = null;
    let id = null;
    let synopsis = null;
    let imageUrl = null;
    let earthOrbitAngle = 0;
    const AU = 10;
    const daysInYear = 365.25;
    const degreesPerDay = 360 / daysInYear;

    onMount(async () => {
        const hashFragment = window.location.hash.substring(1);
        id = hashFragment.split("/").pop();

        try {
            const response = await axios.get("/data/stardata.json");
            const jsonData = response.data;
            data = jsonData.filter((s) => s.proper);
            star = jsonData.find((s) => s.id === parseInt(id, 10));

            if (!star) {
                throw new Error("Star not found");
            }

            if (star.wikiUrl) {
                synopsis = await fetchStarSynopsis(star);
                imageUrl = await fetchStarImage(star);
            }

            const sunData = jsonData.find((s) => s.proper === "Sol");
            if (sunData) {
                sunPosition = calculatePosition(sunData.ra, sunData.dec);
            }

            animateEarth();
        } catch (err) {
            console.error("Error fetching data:", err);
            error = err.message;
        }
    });

    function goBack() {
        push("/");
    }

    async function fetchStarSynopsis(star) {
        if (!star.wikiUrl) {
            throw new Error("Star does not have a wiki_url key");
        }

        const wikiApiUrl = `https://en.wikipedia.org/api/rest_v1/page/summary/${star.wikiUrl.split("/").pop()}`;

        try {
            const response = await axios.get(wikiApiUrl);
            return response.data.extract;
        } catch (error) {
            console.error("Error fetching Wikipedia synopsis:", error);
            return null;
        }
    }

    async function fetchStarImage(star) {
        if (!star.wikiUrl) {
            throw new Error("Star does not have a wiki_url key");
        }

        const wikiApiUrl = `https://en.wikipedia.org/api/rest_v1/page/summary/${star.wikiUrl.split("/").pop()}`;

        try {
            const response = await axios.get(wikiApiUrl);
            return response.data.thumbnail?.source || null;
        } catch (error) {
            console.error("Error fetching Wikipedia image:", error);
            return null;
        }
    }

    function calculatePosition(ra, dec) {
        const x = (ra / 24) * 100;
        const y = ((dec + 90) * 100) / 180;
        return { x, y };
    }

    let starPosition = { x: 0, y: 0 };
    let sunPosition = { x: 50, y: 50 };
    const earthOrbitRadius = AU;

    $: if (star) {
        starPosition = calculatePosition(star.ra, star.dec);
    }

    function animateEarth() {
        earthOrbitAngle = (earthOrbitAngle + degreesPerDay / 365.25) % 360;
        tick().then(() => requestAnimationFrame(animateEarth));
    }

    function calculateEarthPosition(angle) {
        return {
            x:
                sunPosition.x +
                earthOrbitRadius * Math.cos((angle * Math.PI) / 180),
            y:
                sunPosition.y +
                earthOrbitRadius * Math.sin((angle * Math.PI) / 180),
        };
    }

    $: earthPosition = calculateEarthPosition(earthOrbitAngle);
</script>

<main>
    <div class="container">
        <button class="back-button" on:click={goBack}>Back to Home</button>
        {#if error}
            <p class="error">Error: {error}</p>
        {:else if star}
            <div>
                <h1>{star.proper}</h1>
                <h2>{star.con}</h2>
                <p><strong>Magnitude:</strong> {star.mag}</p>
                <p>
                    <strong>Coordinates:</strong> (x: {star.x0}, y: {star.y0},
                    z: {star.z0})
                </p>
                <p><strong>Distance:</strong> {star.dist} pc</p>
                <p><strong>Spectral Type:</strong> {star.spect}</p>
                <p><strong>Color Index (CI):</strong> {star.ci}</p>
                {#if star.rv}
                    <p><strong>Radial Velocity:</strong> {star.rv} km/s</p>
                {/if}
                {#if synopsis}
                    <p><strong>Synopsis:</strong> {synopsis}</p>
                {/if}
                {#if imageUrl}
                    <img src={imageUrl} alt="Star image" />
                {/if}
                <div class="sky-map">
                    <p>Position on the sky:</p>
                    <svg class="star-chart" viewBox="0 0 100 100">
                        {#each data as s}
                            <circle
                                cx={calculatePosition(s.ra, s.dec).x}
                                cy={calculatePosition(s.ra, s.dec).y}
                                r={s.id === star.id ? "2" : "0.5"}
                                fill={s.id === star.id ? "orange" : "gray"}
                            />
                        {/each}
                        <circle
                            cx={sunPosition.x}
                            cy={sunPosition.y}
                            r="2.5"
                            fill="yellow"
                        />
                        <circle
                            cx={earthPosition.x}
                            cy={earthPosition.y}
                            r="1"
                            fill="blue"
                        />
                    </svg>
                </div>
                {#if star.wikiUrl}
                    <p><a href={star.wikiUrl} target="_blank">Wikipedia</a></p>
                {/if}
            </div>
        {:else}
            <p>Loading...</p>
        {/if}
    </div>
</main>

<style>
    main {
        font-family: "Roboto", sans-serif;
        padding: 20px;
        background-color: #151531;
        color: #f2fdff;
        min-height: 100vh;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .container {
        width: 100%;
        max-width: 800px;
        background-color: #1e1e4a;
        padding: 20px;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
    }

    .back-button {
        width: 100%;
        padding: 15px;
        font-size: 16px;
        font-weight: 500;
        cursor: pointer;
        background-color: #6f9ceb;
        color: #f2fdff;
        border: none;
        border-radius: 5px;
        margin-bottom: 20px;
        transition:
            background-color 0.3s,
            box-shadow 0.3s;
    }

    .back-button:hover {
        background-color: #5177b9;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
    }

    h1 {
        font-size: 24px;
        margin-bottom: -17px;
        color: #f2fdff;
    }

    h2 {
        font-size: 18px;
        margin-bottom: 20px;
        color: #6f9ceb;
    }

    p {
        font-size: 16px;
        margin-bottom: 10px;
    }

    a {
        color: #6f9ceb;
        text-decoration: none;
    }

    a:hover {
        text-decoration: underline;
    }

    .error {
        color: #ff4a4a;
    }

    img {
        max-width: 100%;
        border-radius: 8px;
        margin-top: 10px;
    }

    .sky-map {
        margin-top: 20px;
    }

    .star-chart {
        width: 80%;
        max-width: 400px;
        height: 300px;
        background: #0d1b2a;
        border: 1px solid #6f9ceb;
        border-radius: 8px;
        position: relative;
        margin: 0 auto;
    }
</style>
