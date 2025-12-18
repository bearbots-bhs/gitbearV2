<script>
    import { fade, slide } from 'svelte/transition';
    import { onMount } from "svelte";
    import { base } from "$app/paths";

    let toggle = $state(false);
    let loadMessage = $state("Putting those two braincells to work...")

    onMount(() => {
        setTimeout(() => {toggle = true}, 100);
    })

    let archive = $state([]);
    let archiveSorted = $state([]);
 

    onMount(async() => {
        let curr = 2024;
        let end = false;
        while (true) {
            let yearCheck = await fetch(`https://raw.githubusercontent.com/bearbots-bhs/${curr}/refs/heads/main/config.json`);
            if (!yearCheck.ok) {
                if (archive.length > 0) {
                    loadMessage = `Found ${archive.length} results`;
                }
                break;
            }

            let yearData = await fetch(`https://raw.githubusercontent.com/bearbots-bhs/${curr}/refs/heads/main/config.json`);
            if (yearData.ok) {
                let data = await yearData.json();
                let name = data[0].name;
                let length = data[1].length;
                let download = `https://github.com/bearbots-bhs/${curr}/archive/refs/heads/main.zip`
                archive.push([name, curr, download, length]);
            }
            else {
                if (archive.length > 0) {
                    loadMessage = "Uh oh; there was an error ;-;";
                }
                break;
            }
            curr++;
        }
        //console.log(archive);
        archiveSorted = [];
        for (let x = archive.length-1; x >= 0; x--) {
            archiveSorted.push(archive[x])
        }

    })
</script>
<style>
    #titleSpan {
        color: rgb(255, 174, 0);
    }

    .season {
        position: relative;
        background-color: rgb(71, 71, 146);
        padding: 10px;
        border-radius: 20px;
        margin: 10px;
        margin-bottom: 20px;
        overflow: hidden;

        h2 { 
            user-select: none; 
            -webkit-user-select: none; 
            -moz-user-select: none; 
            -ms-user-select: none; 
        } 
        h2 span { 
            background-color: rgb(115, 132, 156); 
            border-radius: 5px; 
            padding: 5px; 
        } 
        h3 { 
            user-select: none; 
            -webkit-user-select: none; 
            -moz-user-select: none; 
            -ms-user-select: none; 
        } 
        h4 { 
            user-select: none; 
            -webkit-user-select: none; 
            -moz-user-select: none; 
            -ms-user-select: none; 
        }
    }

    .season::after {
        content: "";
        position: absolute;
        inset: 0;
        background-image: repeating-linear-gradient(
            30deg,
            rgba(71, 71, 146, 0),
            rgba(71, 71, 146, 0) 25px,
            rgba(94, 94, 190, 0) 25px,
            rgba(94, 94, 190, 0) 50px
        );
        z-index: 0;
    }

    .season::before {
        content: "";
        position: absolute;
        inset: 0;
        background-image: repeating-linear-gradient(
            30deg,
            rgb(71, 71, 146),
            rgb(71, 71, 146) 25px,
            rgb(94, 94, 190) 25px,
            rgb(94, 94, 190) 50px
        );
        opacity: 0;
        transition: opacity 0.5s ease;
        z-index: 0;
    }

    .season:hover::before {
        opacity: 1;
    }

    .season > * {
        position: relative;
        z-index: 1;
    }

</style>

{#if toggle}
    <h1 transition:slide style="user-select: none;"><span id="titleSpan">BEARBOTS</span> ONBOT JAVA ARCHIVE</h1>
    <h3 style="margin-top: 80px; margin-bottom: 50px;" transition:fade={{delay: 500}}>{loadMessage}</h3>
    {#each archiveSorted as x}
        <div class="season">
            <h2>{x[0]}  <span>{x[1]}/{x[1]+1}</span></h2>
            <h4><i>Contains {x[3]} scripts</i></h4>
            <h3><button onclick = {() => {window.location.href = base + `/view/${x[1]}`}}><span translate="no" title="View Scripts" class="material-symbols-outlined">code_blocks</span></button><button onclick={() => {window.location.href = `https://github.com/bearbots-bhs/${x[1]}/archive/refs/heads/main.zip`}}><span translate="no" title="Download Scripts" class="material-symbols-outlined">download</span></button><button onclick={() => {window.location.href = `https://github.com/bearbots-bhs/${x[1]}`}}><span translate="no" title="View GitHub Repository" class="material-symbols-outlined">data_object</span></button></h3>
        </div>
    {/each}
{/if}