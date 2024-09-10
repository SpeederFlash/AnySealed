<script>
    import { onMount } from "svelte";
    import { browser } from "$app/environment";

    /** @type {Object.<string,Object>} */
    let draftingBoosters = {};

    /** @type {Object.<string,string>} - UUID, ScryfallAPIID */
    let uuidToScryfall = {};

    /** @type {Object.<string,string>} Identity */
    let uuidToColorIdentity = {};

    /** @type {Object} */
    let data;

    /** @type {Array<string>} */
    let cards = [];

    /** @type {Object.<string,string>} */
    let uuidToImage = {};

    /**
     * Handles a click to another page
     * @param {string} route - Route to another local page
     */
    function handlePageTransfer(route) {
        window.location.href = route;
    }

    /**
     * Generate a magic pack
     * @param {string}code
     */
    async function generatePack(code) {
        /** @type {any} - JSDOC don't wanna be nice. It has a Object, I just can't be bothered to get it to play nice */
        var boosterPackSpecs = draftingBoosters[code];
        // Choose booster by weight
        var chosen =
            Math.floor(Math.random() * boosterPackSpecs.boostersTotalWeight) +
            1;
        var currentPackI = 0;
        for (let i = 0; i < boosterPackSpecs.boosters.length; i++) {
            chosen -= boosterPackSpecs.boosters[i].weight;
            if (chosen <= 0) {
                currentPackI = i;
                break;
            }
        }
        var currentPack = boosterPackSpecs.boosters[currentPackI];
        // Card by sheet by booster specs
        let entries = Object.entries(currentPack.contents);
        for (var i in entries) {
            if (entries[i][0] === "theList") {
                continue;
            }
            var sheet = boosterPackSpecs.sheets[entries[i][0]];

            for (let times = 0; times < entries[i][1]; times++) {
                var card_chosen =
                    Math.floor(Math.random() * sheet.totalWeight) + 1;
                var card_i = "";
                for (var key in sheet.cards) {
                    card_chosen -= sheet.cards[key];
                    if (card_chosen <= 0) {
                        card_i = key;
                        break;
                    }
                }
                cards.push(card_i);
            }
        }
        cards = cards;
    }

    async function getMTJSON() {
        let boosters = Object.entries(data);
        // CHECKS FOR THE SPECIAL SLOTS
        if (boosters.filter((e) => e[0] === "OTJ").length > 0) {
            boosters.push(["OTP", 1]);
        }
        if (boosters.length != 0) {
            for (var b in boosters) {
                let url =
                    "https://mtgjson.com/api/v5/" + boosters[b][0] + ".json";
                let f = await fetch(url);
                let t_data = (await f.json()).data;
                if (t_data.booster !== undefined) {
                    if (t_data.booster.play != undefined) {
                        draftingBoosters[boosters[b][0]] = t_data.booster.play;
                    } else if (t_data.booster.draft != undefined) {
                        draftingBoosters[boosters[b][0]] = t_data.booster.draft;
                    } else {
                        draftingBoosters[boosters[b][0]] =
                            t_data.booster.default;
                    }
                }
                // TAKE CARDS
                for (var x in t_data.cards) {
                    var card = t_data.cards[x];
                    uuidToScryfall[card.uuid] = card.identifiers.scryfallId;
                    if (uuidToColorIdentity[card.uuid] === undefined) {
                        for (var e in card.colorIdentity) {
                            if (uuidToColorIdentity[card.uuid] !== undefined) {
                                uuidToColorIdentity[card.uuid] =
                                    uuidToColorIdentity[card.uuid] +
                                    card.colorIdentity[e];
                            } else {
                                uuidToColorIdentity[card.uuid] =
                                    card.colorIdentity[e];
                            }
                        }
                    }
                }
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

    /**
     * Sleep for time milliseconds
     * @param {number} time
     */
    const sleep = (time) => new Promise((r) => setTimeout(r, time));

    async function getImages() {
        var cardsLeft = true;
        var currIndex = 0;
        while (cardsLeft) {
            let sfCards = cards.slice(currIndex, currIndex + 75);
            currIndex += 75;
            if (currIndex > cards.length) {
                cardsLeft = false;
            }
            /** @type {{identifiers: {id: string}[]}} */
            let tempObject = {};
            tempObject.identifiers = [];
            for (let i = 0; i < 75; i++) {
                tempObject.identifiers.push({ id: "" });
            }
            for (let i = 0; i < 75; i++) {
                tempObject.identifiers[i].id = uuidToScryfall[sfCards[i]];
            }
            tempObject.identifiers = tempObject.identifiers.filter((elem) => {
                return elem.id !== undefined;
            });
            let cardsReturned = await (
                await fetch("https://api.scryfall.com/cards/collection", {
                    method: "POST",
                    body: JSON.stringify(tempObject),
                    headers: {
                        "Content-Type": "application/json",
                    },
                })
            ).json();
            await sleep(100);
            for (let img_i = 0; img_i < cardsReturned.data.length; img_i++) {
                uuidToImage[sfCards[img_i]] =
                    cardsReturned.data[img_i].image_uris.normal;
            }
        }
    }

    onMount(async function () {
        await getCookies();
        await getMTJSON();
        var entries = Object.entries(data);
        for (var entry in entries) {
            for (var i = 0; i < entries[entry][1]; i++) {
                await generatePack(entries[entry][0]);
            }
        }
        await getImages();
    });
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
</div>

<div id="imageHolder">
    <div class="identityDiv">
        {#each cards.filter((e) => {
            return uuidToColorIdentity[e] === "W";
        }) as card}
            <img src={uuidToImage[card]} alt="v" />
        {/each}
    </div>
    <div class="identityDiv">
        {#each cards.filter((e) => {
            return uuidToColorIdentity[e] === "U";
        }) as card}
            <img src={uuidToImage[card]} alt="v" />
        {/each}
    </div>
    <div class="identityDiv">
        {#each cards.filter((e) => {
            return uuidToColorIdentity[e] === "B";
        }) as card}
            <img src={uuidToImage[card]} alt="v" />
        {/each}
    </div>
    <div class="identityDiv">
        {#each cards.filter((e) => {
            return uuidToColorIdentity[e] === "R";
        }) as card}
            <img src={uuidToImage[card]} alt="v" />
        {/each}
    </div>
    <div class="identityDiv">
        {#each cards.filter((e) => {
            return uuidToColorIdentity[e] === "G";
        }) as card}
            <img src={uuidToImage[card]} alt="v" />
        {/each}
    </div>
    <div class="identityDiv">
        {#each cards.filter((e) => {
            return uuidToColorIdentity[e] === undefined;
        }) as card}
            <img src={uuidToImage[card]} alt="v" />
        {/each}
    </div>
    <div class="identityDiv">
        {#each cards.filter((e) => {
            if (uuidToColorIdentity[e] !== undefined) {
                return uuidToColorIdentity[e].length > 1;
            } else {
                return false;
            }
        }) as card}
            <img src={uuidToImage[card]} alt={uuidToColorIdentity[card]} />
        {/each}
    </div>
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

    .identityDiv {
        width: 10%;
    }

    #imageHolder {
        display: flex;
        flex-direction: row;
    }

    img {
        width: 100%;
    }
</style>
