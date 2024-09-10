<script>
    import { browser } from "$app/environment";

    /** @type {Object.<string,number>} */
    export let selectedBoosters = {};

    /** @type {Array.<Object.<string,string>>} */
    let validBoosters = [
        {
            code: "OTJ",
            text: "Outlaws of Thunder Junction",
        },
        {
            code: "MKM",
            text: "Murders at Karlov Manor",
        },
        {
            code: "BLB",
            text: "Bloomburrow",
        },
    ];
    /** @type {Object.<string,string>} */
    let selected;

    /** @type {HTMLElement} */
    let removeEntry;

    /**
     * Handles a click to another page
     * @param {string} route - Route to another local page
     */
    function handlePageTransfer(route) {
        window.location.href = route;
    }

    /**
     * Adds to booster list
     * @param {string | null} boosterCode - Booster pack set code
     * @param {number} amount - Default 1, amount to add
     */
    function addToBoosterList(boosterCode, amount = 1) {
        if (boosterCode === null) {
            return;
        }
        if (selectedBoosters[boosterCode] === undefined) {
            selectedBoosters[boosterCode] = 1;
        } else {
            if (selectedBoosters[boosterCode] + amount > 0) {
                selectedBoosters[boosterCode] += amount;
            } else {
                delete selectedBoosters[boosterCode];
            }
        }
        selectedBoosters = selectedBoosters;
    }

    function redirectToRun() {
        if (Object.entries(selectedBoosters).length === 0) {
            alert("No boosters selected");
        } else {
            browser &&
                sessionStorage.setItem(
                    "boosters",
                    JSON.stringify(selectedBoosters),
                );
            handlePageTransfer("./run");
        }
    }
</script>

<div id="underHeader"></div>
<div id="headerDiv">
    <h1>AnySealed</h1>
    <button
        class="headerButton button"
        id="aboutButton"
        on:click={() => {
            handlePageTransfer("./about");
        }}>About</button
    >
</div>

<div class="centeredDiv" id="selectorDiv">
    <select bind:value={selected} id="boosters">
        {#each validBoosters as boosterObj}
            <option value={boosterObj}>{boosterObj.text}</option>
        {/each}
    </select>
    <div class="buffer" style:--div-width="50px" style:--div-height="0px"></div>
    <button
        class="button"
        on:click={() => addToBoosterList(selected ? selected.code : null)}
        >Add</button
    >
</div>
<div class="buffer" style:--div-width="100%" style:--div-height="20px"></div>
<div class="centeredDiv">
    <div class="centeredDiv selector" id="listDiv">
        {#each Object.entries(selectedBoosters) as [boost, n]}
            <div class="flexRow">
                <li class="listEntry">{boost} {n}</li>
                <button
                    class="button listButton"
                    on:click={() => addToBoosterList(boost, -1)}>Remove</button
                >
            </div>
        {/each}
    </div>
</div>
<div class="buffer" style:--div-width="100%" style:--div-height="20px"></div>
<div class="centeredDiv">
    <button class="button runButton" on:click={() => redirectToRun()}
        >Run</button
    >
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

    .buffer {
        width: var(--div-width);
        height: var(--div-height);
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

    .button {
        position: relative;
        font-family: "Rubik Mono One", monospace;
        font-weight: 400;
        font-style: normal;
        font-size: 10px;
        padding: 10px;
        text-align: center;
        color: #c8acd6;
        text-shadow: -3px -3px 0 #433d8b;
        background: transparent;
        border: 2px solid transparent;
        border-radius: 10px;
    }

    .button:hover {
        border: 2px solid #c8acd6;
        border-color: #c8acd6;
    }

    select {
        font-family: "Rubik Mono One", monospace;
        font-weight: 400;
        font-style: normal;
        text-align: center;
        color: #c8acd6;
        text-shadow: -3px -3px 0 #433d8b;
        background-color: #00000020;
        border: 2px solid #c8acd6;
        border-radius: 10px;
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
        border: 5px solid #c8acd6;
        border-color: #c8acd6;
    }

    .runButton {
        position: relative;
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

    .runButton:hover {
        border: 5px solid #c8acd6;
        border-color: #c8acd6;
    }

    #selectorDiv {
        width: 100%;
    }

    .centeredDiv {
        position: relative;
        display: flex;
        justify-content: center;
    }

    .selector {
        font-family: "Rubik Mono One", monospace;
        font-weight: 400;
        font-style: normal;
        text-align: center;
        color: #c8acd6;
        text-shadow: -3px -3px 0 #433d8b;
        flex-direction: column;
        justify-content: flex-start;
    }

    li {
        list-style-type: none;
        align-items: center;
    }

    .listEntry {
        display: flex;
        flex-direction: row;
        width: 80%;
    }

    .listButton {
        display: flex;
        align-self: flex-end;
        align-content: center;
    }

    .flexRow {
        display: flex;
        flex-direction: row;
    }

    #listDiv {
        padding: 50px;
        border-radius: 24px;
        width: 20%;
        height: 200px;
        background-color: #00000020;
        overflow: scroll;
    }
</style>
