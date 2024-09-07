<script>
    import { onMount } from "svelte";
    import { browser } from "$app/environment";

    /** @type {Object.<string,Array<Object>>} */
    let draftingBoosters = {};

    /** @type {Object.<string,string>} - UUID, ScryfallAPIID */
    let uuidToScryfall = {};

    /** @type {Object} */
    let data;
    /**
     * Handles a click to another page
     * @param {string} route - Route to another local page
     */
    function handlePageTransfer(route) {
        window.location.href = route;
    }

    async function getMTJSON() {
        let boosters = Object.entries(data);
        if (boosters.length != 0) {
            for (var b in boosters) {
                let url =
                    "https://mtgjson.com/api/v5/" + boosters[b][0] + ".json";
                let f = await fetch(url);
                let t_data = (await f.json()).data;
                if (t_data.booster.play != undefined) {
                    draftingBoosters[boosters[b][0]] = [
                        ...t_data.booster.play.boosters,
                    ];
                } else if (t_data.booster.draft != undefined) {
                    draftingBoosters[boosters[b][0]] = [
                        ...t_data.booster.draft.boosters,
                    ];
                } else {
                    draftingBoosters[boosters[b][0]] = [
                        ...t_data.booster.default.boosters,
                    ];
                }
                // TAKE CARDS
                for (var x in t_data.cards) {
                    var card = t_data.cards[x];
                    uuidToScryfall[card.uuid] = card.identifiers.scryfallId;
                }
                console.log(uuidToScryfall);
                t_data = undefined;
            }
        }
    }

    async function getCookies() {
        if (browser === false) {
            data = {};
        } else {
            /** @type {string | null} */
            let temp = sessionStorage.getItem("boosters");
            if (temp === null) {
                temp = "{}";
            }
            data = JSON.parse(temp);
        }
    }

    onMount(async function () {
        await getCookies();
        await getMTJSON();
    });

    function refresh() {
        draftingBoosters = draftingBoosters;
    }
</script>

<div id="underHeader"></div>
<div id="headerDiv">
    <h1>RUN</h1>
    <button
        class="headerButton"
        id="homeButton"
        on:click={() => {
            handlePageTransfer("/");
        }}>Home</button
    >
    <button
        class="headerButton"
        on:click={() => {
            refresh();
        }}>Refresh</button
    >
</div>

<div>
    {#each Object.entries(uuidToScryfall) as [code, booster]}
        <p>{code} {booster}</p>
    {/each}
</div>

<style>
    @font-face {
        font-family: "Rubik Mono One";
        font-style: normal;
        font-weight: 400;
        font-display: swap;
        src: url(https://fonts.gstatic.com/s/rubikmonoone/v18/UqyJK8kPP3hjw6ANTdfRk9YSN985TKUbcw.woff2)
            format("woff2");
        unicode-range: U+0100-02AF, U+0304, U+0308, U+0329, U+1E00-1E9F,
            U+1EF2-1EFF, U+2020, U+20A0-20AB, U+20AD-20C0, U+2113, U+2C60-2C7F,
            U+A720-A7FF;
    }

    @font-face {
        font-family: "Rubik Mono One";
        font-style: normal;
        font-weight: 400;
        font-display: swap;
        src: url(https://fonts.gstatic.com/s/rubikmonoone/v18/UqyJK8kPP3hjw6ANTdfRk9YSN983TKU.woff2)
            format("woff2");
        unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6,
            U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+2074, U+20AC,
            U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
    }

    #headerDiv {
        position: fixed;
        top: 0;
        left: 0;
        box-sizing: border-box;
        width: 100%;
        height: 180px;
        border-radius: 0px 0px 24px 24px;
        border: 5px solid #c8acd6;
        background: #2e236c;
        z-index: 100;
    }
    #underHeader {
        position: relative;
        top: 0;
        left: 0;
        padding: 0;
        margin: 0;
        border: 0;
        background: transparent;
        width: 100%;
        height: 180px;
    }

    :global(body) {
        background: #17153b;
    }

    h1 {
        font-family: "Rubik Mono One", monospace;
        font-weight: 400;
        font-style: normal;
        font-size: 50px;
        text-align: center;
        color: #c8acd6;
        text-shadow: -3px -3px 0 #433d8b;
    }

    p {
        font-family: "Rubik Mono One", monospace;
        font-weight: 400;
        font-style: normal;
        font-size: 15px;
        text-align: center;
        color: #c8acd6;
        text-shadow: -3px -3px 0 #433d8b;
    }

    .headerButton {
        position: relative;
        bottom: 20%;
        left: 15%;
        font-family: "Rubik Mono One", monospace;
        font-weight: 400;
        font-style: normal;
        font-size: 25px;
        padding: 15px;
        text-align: center;
        color: #c8acd6;
        text-shadow: -3px -3px 0 #433d8b;
        background: transparent;
        border: 5px solid transparent;
        border-radius: 10px;
    }

    .headerButton:hover {
        padding: 15px;
        border: 5px solid #c8acd6;
        border-color: #c8acd6;
    }
</style>
