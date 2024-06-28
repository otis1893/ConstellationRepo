<script>
    import { onMount } from "svelte";
    import { push } from "svelte-spa-router";
    import axios from "axios";
    import { writable } from "svelte/store";

    let data = [];
    let error = null;
    let constellationCounts = [];
    let selectedConstellation = writable("");
    let sliderValue = writable(0);

    const constellationNames = {
        And: "Andromeda",
        Ant: "Antlia",
        Aps: "Apus",
        Aqr: "Aquarius",
        Aql: "Aquila",
        Ara: "Ara",
        Ari: "Aries",
        Aur: "Auriga",
        Boo: "Boötes",
        Cae: "Caelum",
        Cam: "Camelopardalis",
        Cnc: "Cancer",
        CVn: "Canes Venatici",
        CMa: "Canis Major",
        CMi: "Canis Minor",
        Cap: "Capricornus",
        Car: "Carina",
        Cas: "Cassiopeia",
        Cen: "Centaurus",
        Cep: "Cepheus",
        Cet: "Cetus",
        Cha: "Chamaeleon",
        Cir: "Circinus",
        Col: "Columba",
        Com: "Coma Berenices",
        CrA: "Corona Australis",
        CrB: "Corona Borealis",
        Crv: "Corvus",
        Crt: "Crater",
        Cru: "Crux",
        Cyg: "Cygnus",
        Del: "Delphinus",
        Dor: "Dorado",
        Dra: "Draco",
        Equ: "Equuleus",
        Eri: "Eridanus",
        For: "Fornax",
        Gem: "Gemini",
        Gru: "Grus",
        Her: "Hercules",
        Hor: "Horologium",
        Hya: "Hydra",
        Hyi: "Hydrus",
        Ind: "Indus",
        Lac: "Lacerta",
        Leo: "Leo",
        LMi: "Leo Minor",
        Lep: "Lepus",
        Lib: "Libra",
        Lup: "Lupus",
        Lyn: "Lynx",
        Lyr: "Lyra",
        Men: "Mensa",
        Mic: "Microscopium",
        Mon: "Monoceros",
        Mus: "Musca",
        Nor: "Norma",
        Oct: "Octans",
        Oph: "Ophiuchus",
        Ori: "Orion",
        Pav: "Pavo",
        Peg: "Pegasus",
        Per: "Perseus",
        Phe: "Phoenix",
        Pic: "Pictor",
        Psc: "Pisces",
        PsA: "Piscis Austrinus",
        Pup: "Puppis",
        Pyx: "Pyxis",
        Ret: "Reticulum",
        Sge: "Sagitta",
        Sgr: "Sagittarius",
        Sco: "Scorpius",
        Scl: "Sculptor",
        Sct: "Scutum",
        Ser: "Serpens",
        Sex: "Sextans",
        Tau: "Taurus",
        Tel: "Telescopium",
        Tri: "Triangulum",
        TrA: "Triangulum Australe",
        Tuc: "Tucana",
        UMa: "Ursa Major",
        UMi: "Ursa Minor",
        Vel: "Vela",
        Vir: "Virgo",
        Vol: "Volans",
        Vul: "Vulpecula",
    };

    onMount(async () => {
        try {
            const response = await axios.get("/data/stardata.json");
            const jsonData = response.data;
            data = jsonData.filter((star) => star.proper);

            const constellationMap = new Map();
            data.forEach((star) => {
                const con = star.con;
                if (constellationMap.has(con)) {
                    constellationMap.set(con, constellationMap.get(con) + 1);
                } else {
                    constellationMap.set(con, 1);
                }
            });

            // Konstellationen nach Anzahl der Sterne sortieren
            constellationCounts = Array.from(constellationMap.entries()).sort(
                (a, b) => b[1] - a[1],
            );
        } catch (err) {
            console.error("Error fetching data:", err);
            error = err.message;
        }
    });

    function handleButtonClick(id) {
        push(`/detail/${id}`);
    }

    function sortStarsByMagnitude(stars, targetMagnitude) {
        return stars.sort(
            (a, b) =>
                Math.abs(a.mag - targetMagnitude) -
                Math.abs(b.mag - targetMagnitude),
        );
    }

    $: filteredStars =
        $selectedConstellation === ""
            ? data
            : data.filter((star) => star.con === $selectedConstellation);

    // Setze den Zielwert der Helligkeit basierend auf dem Schiebereglerwert (linear)
    $: targetMagnitude = 0 + (4 - 0) * $sliderValue;

    $: sortedStars = sortStarsByMagnitude(
        filteredStars.filter(
            (star) => star.mag <= targetMagnitude || star.mag < 0,
        ),
        targetMagnitude,
    );
</script>

<main>
    {#if error}
        <p class="error">{error}</p>
    {:else if data.length > 0}
        <div class="container">
            <h1>Star Data</h1>
            <label for="constellation">Filter by Constellation:</label>
            <select
                id="constellation"
                on:change={(e) => selectedConstellation.set(e.target.value)}
            >
                <option value="">All Constellations</option>
                {#each constellationCounts as [constellation, count]}
                    <option value={constellation}
                        >{constellationNames[constellation]} ({count} stars)</option
                    >
                {/each}
            </select>
            <label for="magnitudeSlider">Adjust Brightness:</label>
            <div class="slider-container">
                <input
                    type="range"
                    id="magnitudeSlider"
                    min="0"
                    max="1"
                    step="0.01"
                    bind:value={$sliderValue}
                />
                <div class="slider-labels">
                    <span>0</span>
                    <span>1</span>
                    <span>2</span>
                    <span>3</span>
                    <span>4</span>
                </div>
            </div>
            <ul>
                {#each sortedStars.slice(0, 1000) as star (star.id)}
                    <li>
                        <button
                            on:click={() => handleButtonClick(star.id)}
                            id={"button-" + star.id}
                        >
                            <span class="name">{star.proper}</span>
                            <span class="info"
                                >Mag: {star.mag} | Dist: {star.dist} pc | Con: {star.con}</span
                            >
                        </button>
                    </li>
                {/each}
            </ul>
        </div>
    {:else}
        <p>Loading...</p>
    {/if}
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

    h1 {
        margin-bottom: 20px;
        font-size: 24px;
        text-align: center;
        color: #f2fdff;
    }

    label {
        font-size: 16px;
        margin-bottom: 10px;
        display: block;
    }

    select {
        width: 100%;
        padding: 10px;
        font-size: 16px;
        margin-bottom: 40px;
        margin-top: 5px;
        border-radius: 5px;
        border: none;
        background-color: #6f9ceb;
        color: white;
        cursor: pointer;
        outline: none;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
        transition:
            background-color 0.3s,
            box-shadow 0.3s;
    }

    select:hover {
        background-color: #5177b9;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
    }

    .slider-container {
        width: 70%; /* Reduziere die Breite des Schiebereglers */
        margin: 0 auto;
        text-align: center;
        padding-bottom: 30px;
        padding-top: 30px;
    }

    input[type="range"] {
        width: 100%;
        padding: 10px;
        margin-bottom: 10px;
        border-radius: 5px;
        border: none;
        background-color: #6f9ceb;
        cursor: pointer;
        outline: none;
        transition:
            background-color 0.3s,
            box-shadow 0.3s;
    }

    input[type="range"]:hover {
        background-color: #266789;
    }

    .slider-labels {
        display: flex;
        justify-content: space-between;
        padding: 0 10px;
        color: #f2fdff;
    }

    ul {
        list-style-type: none;
        padding: 0;
        margin: 0;
    }

    li {
        margin: 10px 0;
    }

    button {
        width: 100%;
        padding: 15px;
        font-size: 16px;
        font-weight: 500;
        cursor: pointer;
        background-color: #6f9ceb;
        color: #f2fdff;
        border: none;
        border-radius: 5px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        transition:
            background-color 0.3s,
            box-shadow 0.3s;
    }

    button:hover {
        background-color: #5177b9;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
    }

    .name {
        font-weight: 700;
    }

    .info {
        font-weight: 400;
        font-size: 14px;
        color: #f2fdff;
    }

    .error {
        color: #ff4a4a;
    }
</style>
