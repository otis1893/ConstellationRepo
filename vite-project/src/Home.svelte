<script>
    import { onMount } from "svelte";
    import { push } from "svelte-spa-router";
    import axios from "axios";
    import { writable } from "svelte/store";

    let data = [];
    let error = null;
    let constellationCounts = [];
    let selectedStar = null;
    let hoveredStar = null;
    let mouseX = 0;
    let mouseY = 0;
    let selectedConstellation = writable("");
    let sliderValue = writable(1); // Initial value set to 1 (maximum)
    let starOutOfRange = writable(false);
    let searchQuery = writable("");
    let meteors = writable([]);

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

    const constellationLines = {
        And: [
            // Example format: [[star1, star2], [star2, star3], ...]
            ["star1", "star2"],
            ["star2", "star3"],
        ],
        // Add other constellations here...
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

            constellationCounts = Array.from(constellationMap.entries()).sort(
                (a, b) => b[1] - a[1],
            );

            startMeteorShower();
        } catch (err) {
            console.error("Error fetching data:", err);
            error = err.message;
        }
    });

    function startMeteorShower() {
        setInterval(() => {
            let newMeteor = {
                id: Math.random(),
                x: Math.random() * 80,
                y: Math.random() * 80,
                length: Math.random() * 10 + 5,
                duration: Math.random() * 2 + 1,
            };
            meteors.update((m) => {
                m.push(newMeteor);
                return m;
            });
            setTimeout(() => {
                meteors.update((m) =>
                    m.filter((meteor) => meteor.id !== newMeteor.id),
                );
            }, newMeteor.duration * 1000);
        }, 3000);
    }

    function handleButtonClick(id) {
        push(`/detail/${id}`);
    }

    function handleStarClick(star) {
        selectedStar = star;
        checkStarPosition();
        push(`/detail/${star.id}`);
    }

    function handleStarSelect(star) {
        selectedStar = star;
        checkStarPosition();
    }

    function handleMouseOver(event, star) {
        hoveredStar = star;
        mouseX = event.pageX;
        mouseY = event.pageY;
    }

    function handleMouseMove(event) {
        mouseX = event.pageX;
        mouseY = event.pageY;
    }

    function handleMouseOut() {
        hoveredStar = null;
    }

    function checkStarPosition() {
        const chartSize = 70;
        const position = calculatePosition(selectedStar.ra, selectedStar.dec);
        if (
            position.x < 0 ||
            position.x > chartSize ||
            position.y < 0 ||
            position.y > chartSize
        ) {
            starOutOfRange.set(true);
        } else {
            starOutOfRange.set(false);
        }
    }

    function sortStarsByMagnitude(stars, targetMagnitude) {
        return stars.sort(
            (a, b) =>
                Math.abs(a.mag - targetMagnitude) -
                Math.abs(b.mag - targetMagnitude),
        );
    }

    function calculatePosition(ra, dec) {
        const x = (ra / 24) * 70 + 5;
        const y = ((dec + 90) * 70) / 180 + 5;
        return { x, y };
    }

    $: filteredStars =
        $selectedConstellation === ""
            ? data
            : data.filter((star) => star.con === $selectedConstellation);

    $: targetMagnitude = 0 + (4 - 0) * $sliderValue;

    $: sortedStars = sortStarsByMagnitude(
        filteredStars.filter(
            (star) => star.mag <= targetMagnitude || star.mag < 0,
        ),
        targetMagnitude,
    );

    $: searchedStars = $searchQuery
        ? sortedStars.filter((star) =>
              star.proper.toLowerCase().includes($searchQuery.toLowerCase()),
          )
        : sortedStars;
</script>

<main on:mousemove={handleMouseMove}>
    {#if error}
        <p class="error">{error}</p>
    {:else if data.length > 0}
        <div class="container">
            <h1>Astral Map Explorer</h1>
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
            <div class="star-chart-container">
                <svg class="star-chart" viewBox="0 0 80 80">
                    {#each searchedStars as star}
                        <circle
                            cx={calculatePosition(star.ra, star.dec).x}
                            cy={calculatePosition(star.ra, star.dec).y}
                            r={selectedStar && selectedStar.id === star.id
                                ? "1"
                                : "0.5"}
                            fill={selectedStar && selectedStar.id === star.id
                                ? "#FFE135"
                                : "white"}
                            on:click={() => handleStarClick(star)}
                            on:mouseover={(event) =>
                                handleMouseOver(event, star)}
                            on:mouseout={handleMouseOut}
                        />
                    {/each}
                    {#if $selectedConstellation && constellationLines[$selectedConstellation]}
                        {#each constellationLines[$selectedConstellation] as [star1, star2]}
                            <line
                                x1={calculatePosition(
                                    data.find((s) => s.proper === star1).ra,
                                    data.find((s) => s.proper === star1).dec,
                                ).x}
                                y1={calculatePosition(
                                    data.find((s) => s.proper === star1).ra,
                                    data.find((s) => s.proper === star1).dec,
                                ).y}
                                x2={calculatePosition(
                                    data.find((s) => s.proper === star2).ra,
                                    data.find((s) => s.proper === star2).dec,
                                ).x}
                                y2={calculatePosition(
                                    data.find((s) => s.proper === star2).ra,
                                    data.find((s) => s.proper === star2).dec,
                                ).y}
                                stroke="white"
                                stroke-width="0.2"
                            />
                        {/each}
                    {/if}
                    {#each $meteors as meteor}
                        <line
                            x1={meteor.x}
                            y1={meteor.y}
                            x2={meteor.x + meteor.length}
                            y2={meteor.y + meteor.length}
                            stroke="white"
                            stroke-width="0.5"
                            class="meteor"
                            style="animation-duration: {meteor.duration}s"
                        />
                    {/each}
                </svg>
                {#if $starOutOfRange}
                    <p class="out-of-range">
                        The selected star is out of the current view range.
                    </p>
                {/if}
            </div>
            <div class="search-container">
                <input
                    type="text"
                    placeholder="Search stars by name..."
                    bind:value={$searchQuery}
                />
            </div>
            <ul>
                {#each searchedStars.slice(0, 1000) as star (star.id)}
                    <li>
                        <div
                            class="star-item"
                            on:click={() => handleStarSelect(star)}
                        >
                            <span class="name">{star.proper}</span>
                            <div class="star-info">
                                <span class="info"
                                    >Mag: {star.mag} | Dist: {star.dist} pc | Con:
                                    {star.con}</span
                                >
                            </div>
                            <button
                                on:click={(event) => {
                                    event.stopPropagation();
                                    handleButtonClick(star.id);
                                }}
                                id={"button-" + star.id}
                            >
                                View More
                            </button>
                        </div>
                    </li>
                {/each}
            </ul>
            {#if hoveredStar}
                <div
                    class="tooltip"
                    style="left: {mouseX + 10}px; top: {mouseY + 10}px;"
                >
                    {hoveredStar.proper}
                </div>
            {/if}
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
        letter-spacing: 0.2px;
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

    .search-container {
        width: 800px;
        position: relative;
        margin-bottom: 20px;
    }

    input[type="text"] {
        width: calc(100% - 24px);
        padding: 10px;
        border-radius: 5px;
        border: 2px solid #6f9ceb;
        background-color: #151531;
        color: white;
        outline: none;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
        transition:
            background-color 0.3s,
            box-shadow 0.3s;
    }

    input[type="text"]::placeholder {
        color: #6f9ceb;
    }

    input[type="text"]:hover {
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
    }

    label {
        font-size: 16px;
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
        appearance: none;
        background-repeat: no-repeat;
        background-position: calc(100% - 30px) center;
    }

    select:hover {
        background-color: #5177b9;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
    }

    .slider-container {
        width: 100%;
        margin: 0 auto;
        text-align: center;
        padding-bottom: 20px;
        padding-top: none;
    }

    input[type="range"] {
        width: calc(100% - 20px);
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

    input[type="range"]::-webkit-slider-thumb {
        appearance: none;
        width: 20px;
        height: 20px;
        background-color: #151531;
        border-radius: 50%;
        cursor: pointer;
    }

    input[type="range"]::-moz-range-thumb {
        width: 20px;
        height: 20px;
        background-color: #151531;
        border-radius: 50%;
        cursor: pointer;
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

    .star-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        background-color: #6f9ceb;
        padding: 10px;
        border-radius: 5px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
    }

    .star-info {
        display: flex;
        justify-content: flex-end;
        margin-right: 10px;
        flex-grow: 1;
    }

    .name {
        font-weight: 700;
        margin-right: 20px;
    }

    .info {
        font-weight: 400;
        font-size: 14px;
        color: #f2fdff;
        margin-left: 10px;
        text-align: right;
    }

    button {
        padding: 5px 10px;
        font-size: 14px;
        font-weight: 500;
        cursor: pointer;
        background-color: #1e1e4a;
        color: #f2fdff;
        border: none;
        border-radius: 5px;
        transition:
            background-color 0.3s,
            box-shadow 0.3s;
    }

    button:hover {
        background-color: #5177b9;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
    }

    .star-chart-container {
        position: relative;
        margin-top: 20px;
        width: 100%;
        max-width: 400px;
        margin-left: auto;
        margin-right: auto;
        margin-bottom: 20px;
    }

    .star-chart {
        width: 100%;
        height: 400px;
        background-color: #151531;
        border: 1px solid #6f9ceb;
        border-radius: 8px;
    }

    .meteor {
        animation: moveMeteor linear forwards;
    }

    @keyframes moveMeteor {
        from {
            opacity: 1;
        }
        to {
            opacity: 0;
            transform: translate(50px, 50px);
        }
    }

    .out-of-range {
        position: absolute;
        top: 10px;
        left: 10px;
        color: #151531;
        font-size: 14px;
    }

    .tooltip {
        position: absolute;
        background-color: #151531;
        color: #6f9ceb;
        padding: 5px 10px;
        border-radius: 5px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
        pointer-events: none;
        font-size: 14px;
    }

    .error {
        color: #ff4a4a;
    }
</style>
