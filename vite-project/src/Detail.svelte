<script>
    import { push } from "svelte-spa-router";
    import { onMount, tick } from "svelte";
    import axios from "axios";

    let star = null;
    let error = null;
    let id = null;
    let synopsis = null;
    let imageUrl = null;
    let earthOrbitAngle = 0;
    const AU = 10; // Adjusted scaling factor for Earth's orbit
    const maxDist = 100; // Arbitrary max distance for visualization purposes
    let hoveredConstellation = null; // Variable to store the hovered constellation name
    let tooltipPosition = { x: 0, y: 0 }; // Position of the tooltip

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

    const constellations = [
        {
            name: "Orion",
            stars: [
                { ra: 5.9195, dec: 7.4069 }, // Betelgeuse
                { ra: 5.2423, dec: -8.2016 }, // Rigel
                { ra: 5.5856, dec: 9.9359 }, // Bellatrix
                { ra: 5.9196, dec: -1.2019 }, // Mintaka
                { ra: 5.6036, dec: -1.2019 }, // Alnilam
                { ra: 5.2836, dec: -1.9426 }, // Alnitak
                { ra: 5.4189, dec: 6.3497 }, // Saiph
            ],
        },
        {
            name: "Ursa Major",
            stars: [
                { ra: 11.0621, dec: 61.751 }, // Dubhe
                { ra: 11.0307, dec: 56.3824 }, // Merak
                { ra: 11.8972, dec: 53.6948 }, // Phecda
                { ra: 12.257, dec: 57.0326 }, // Megrez
                { ra: 12.9005, dec: 55.9598 }, // Alioth
                { ra: 13.4204, dec: 54.9254 }, // Mizar
                { ra: 13.7923, dec: 49.3133 }, // Alkaid
            ],
        },
        {
            name: "Scorpius",
            stars: [
                { ra: 16.4901, dec: -26.4319 }, // Antares
                { ra: 15.7341, dec: -25.7275 }, // Dschubba
                { ra: 15.98, dec: -26.114 }, // Acrab
                { ra: 16.3531, dec: -25.5928 }, // Fang
                { ra: 16.0056, dec: -22.6218 }, // Graffias
                { ra: 16.8361, dec: -34.2932 }, // Sargas
                { ra: 17.6217, dec: -37.2951 }, // Shaula
            ],
        },
        {
            name: "Leo",
            stars: [
                { ra: 10.278, dec: 11.9672 }, // Regulus
                { ra: 11.2373, dec: 20.5237 }, // Algieba
                { ra: 11.8177, dec: 14.5719 }, // Zosma
                { ra: 10.3327, dec: 19.8415 }, // Adhafera
                { ra: 10.1223, dec: 16.7627 }, // Algenubi
                { ra: 10.3316, dec: 23.1885 }, // Ras Elased Australis
                { ra: 9.8799, dec: 26.006 }, // Chertan
            ],
        },
        {
            name: "Taurus",
            stars: [
                { ra: 4.5987, dec: 16.5093 }, // Aldebaran
                { ra: 3.7914, dec: 24.1052 }, // Elnath
                { ra: 4.4769, dec: 22.8258 }, // Ain
                { ra: 4.3824, dec: 17.5425 }, // Chamukuy
                { ra: 5.6275, dec: 21.1425 }, // Atlas
                { ra: 5.5334, dec: 24.0914 }, // Pleione
                { ra: 4.329, dec: 15.6277 }, // Prima Hyadum
            ],
        },
        {
            name: "Gemini",
            stars: [
                { ra: 7.5766, dec: 31.8883 }, // Castor
                { ra: 7.7553, dec: 28.0262 }, // Pollux
                { ra: 6.6285, dec: 33.9996 }, // Alhena
                { ra: 7.1856, dec: 21.9823 }, // Wasat
                { ra: 6.247, dec: 22.5068 }, // Mebsuta
                { ra: 7.0697, dec: 20.5703 }, // Mekbuda
                { ra: 6.7321, dec: 25.1311 }, // Tejat Posterior
            ],
        },
        {
            name: "Cassiopeia",
            stars: [
                { ra: 0.6751, dec: 56.5373 }, // Schedar
                { ra: 1.4303, dec: 60.2353 }, // Caph
                { ra: 0.9451, dec: 60.7167 }, // Tsih
                { ra: 0.1507, dec: 59.1498 }, // Ruchbah
                { ra: 1.9066, dec: 63.6702 }, // Segin
                { ra: 2.2934, dec: 57.6456 }, // Achird
            ],
        },
        {
            name: "Cygnus",
            stars: [
                { ra: 20.6905, dec: 45.2803 }, // Deneb
                { ra: 20.3705, dec: 40.2567 }, // Sadr
                { ra: 19.4952, dec: 27.9597 }, // Albireo
                { ra: 21.3097, dec: 30.2271 }, // Gienah
                { ra: 19.77, dec: 35.0833 }, // Rukh
            ],
        },
        // Add more constellations as needed
    ];

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

            if (star.wikiUrl) {
                synopsis = await fetchStarSynopsis(star);
                imageUrl = await fetchStarImage(star);
            }

            // Start the animation
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
            return response.data.extract; // The synopsis of the article
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
            return response.data.thumbnail?.source || null; // The image URL of the article
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
    let sunPosition = { x: 50, y: 50 }; // Center of the chart
    const earthOrbitRadius = AU; // 1 AU is the average distance from Earth to the Sun

    $: if (star) {
        starPosition = calculatePosition(star.ra, star.dec);
        // Use both x and y coordinates for starPosition
        starPosition.x = (star.x0 / maxDist) * 50 + 50;
        starPosition.y = (star.y0 / maxDist) * 50 + 50;
    }

    function animateEarth() {
        earthOrbitAngle = (earthOrbitAngle + 1) % 360;
        tick().then(() => requestAnimationFrame(animateEarth));
    }

    $: earthPosition = {
        x:
            sunPosition.x +
            earthOrbitRadius * Math.cos((earthOrbitAngle * Math.PI) / 180),
        y:
            sunPosition.y +
            earthOrbitRadius * Math.sin((earthOrbitAngle * Math.PI) / 180),
    };

    function handleMouseOver(constellation, event) {
        hoveredConstellation = constellation;
        tooltipPosition = { x: event.pageX, y: event.pageY };
    }

    function handleMouseOut() {
        hoveredConstellation = null;
    }
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
                        {#each constellations as constellation}
                            <g
                                on:mouseover={(event) =>
                                    handleMouseOver(constellation.name, event)}
                                on:mouseout={handleMouseOut}
                            >
                                <rect
                                    x={Math.min(
                                        ...constellation.stars.map(
                                            (s) =>
                                                calculatePosition(s.ra, s.dec)
                                                    .x,
                                        ),
                                    )}
                                    y={Math.min(
                                        ...constellation.stars.map(
                                            (s) =>
                                                calculatePosition(s.ra, s.dec)
                                                    .y,
                                        ),
                                    )}
                                    width={Math.abs(
                                        Math.max(
                                            ...constellation.stars.map(
                                                (s) =>
                                                    calculatePosition(
                                                        s.ra,
                                                        s.dec,
                                                    ).x,
                                            ),
                                        ) -
                                            Math.min(
                                                ...constellation.stars.map(
                                                    (s) =>
                                                        calculatePosition(
                                                            s.ra,
                                                            s.dec,
                                                        ).x,
                                                ),
                                            ),
                                    )}
                                    height={Math.abs(
                                        Math.max(
                                            ...constellation.stars.map(
                                                (s) =>
                                                    calculatePosition(
                                                        s.ra,
                                                        s.dec,
                                                    ).y,
                                            ),
                                        ) -
                                            Math.min(
                                                ...constellation.stars.map(
                                                    (s) =>
                                                        calculatePosition(
                                                            s.ra,
                                                            s.dec,
                                                        ).y,
                                                ),
                                            ),
                                    )}
                                    fill="transparent"
                                />
                                {#each constellation.stars as star, i (star.ra + "-" + star.dec)}
                                    {#if i > 0}
                                        <line
                                            x1={calculatePosition(
                                                constellation.stars[i - 1].ra,
                                                constellation.stars[i - 1].dec,
                                            ).x}
                                            y1={calculatePosition(
                                                constellation.stars[i - 1].ra,
                                                constellation.stars[i - 1].dec,
                                            ).y}
                                            x2={calculatePosition(
                                                star.ra,
                                                star.dec,
                                            ).x}
                                            y2={calculatePosition(
                                                star.ra,
                                                star.dec,
                                            ).y}
                                            stroke="white"
                                            stroke-width="0.2"
                                        />
                                    {/if}
                                    <circle
                                        cx={calculatePosition(star.ra, star.dec)
                                            .x}
                                        cy={calculatePosition(star.ra, star.dec)
                                            .y}
                                        r="0.5"
                                        fill="white"
                                    />
                                {/each}
                            </g>
                        {/each}
                        <circle
                            cx={starPosition.x}
                            cy={starPosition.y}
                            r="2"
                            fill="#f00"
                        />
                        <circle
                            cx={sunPosition.x}
                            cy={sunPosition.y}
                            r="5"
                            fill="yellow"
                        />
                        <circle
                            cx={earthPosition.x}
                            cy={earthPosition.y}
                            r="1"
                            fill="blue"
                        />
                    </svg>
                    {#if hoveredConstellation}
                        <div
                            class="tooltip"
                            style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px"
                        >
                            {hoveredConstellation}
                        </div>
                    {/if}
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
        width: 100%;
        height: 300px;
        background: #0d1b2a;
        border: 1px solid #6f9ceb;
        border-radius: 8px;
        position: relative;
    }

    .tooltip {
        position: absolute;
        background-color: #151531;
        color: #6f9ceb;
        padding: 5px 10px;
        border-radius: 5px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
        font-size: 14px;
        pointer-events: none;
    }
</style>
