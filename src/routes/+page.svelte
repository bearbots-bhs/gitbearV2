<script>
    import { fade, slide } from 'svelte/transition';
    import { onMount } from "svelte";

    let toggle = $state(false);
    let loadMessage = $state("Putting those two braincells to work...")

    onMount(() => {
        setTimeout(() => {toggle = true}, 100);
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