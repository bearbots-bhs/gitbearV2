<script>
    import { fade, slide } from 'svelte/transition';
    import { onMount } from "svelte";

    let toggle = $state(false);
    let loadMessage = $state("Putting those two braincells to work...")

    onMount(() => {
        setTimeout(() => {toggle = true}, 100);
    })

    let archive = $state([]);
    onMount(function() {
        let curr = 2024;
        let end = false;
        while (!end) {
            fetch(`https://api.ftcscout.org/rest/v1/teams/20805/events/${curr}`)
            .then((response) => {
                if (!response.ok) {
                    end = true;
                }
            })
            .then((response) => {
                if (!false) {
                    fetch(`https://raw.githubusercontent.com/bearbots-bhs/${curr}/refs/heads/main/config.json`)
                    .then((response) => response.json())
                    .then((data) => {
                        let name = data[0].name
                        archive.append([curr, name]);
                    })
                }
            })
        }
    })
</script>
<style>
    #titleSpan {
        color: rgb(255, 174, 0);
    }
</style>

{#if toggle}
    <h1 transition:slide style="user-select: none;"><span id="titleSpan">BEARBOTS</span> ONBOT JAVA CODE ARCHIVE</h1>
    <h3 style="margin-top: 100px;" transition:fade={{delay: 500}}>{loadMessage}</h3>
{/if}