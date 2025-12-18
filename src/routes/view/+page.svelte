<script>
    import { fade, slide } from 'svelte/transition';
    import { onMount } from "svelte";
    import { base } from "$app/paths";

    let toggle = $state(false);


    onMount(() => {
        setTimeout(() => {toggle = true}, 100);
        setTimeout(() => {window.location.href = base + "/"}, 2500);
    });
 
</script>
<style>
    h4 span {
        animation: rotate 1s infinite ease-in-out;
    }
    @keyframes rotate {
        0% {
            transform: rotate(0deg);
        }
        50% {
            transform: rotate(180deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }
</style>

{#if toggle}
    <h1 transition:slide style="user-select: none;">RE-DIRECTING</h1>
    <h3 style="margin-top: 10px; margin-bottom: 50px" transition:fade={{delay: 100}}>One moment, please ^w^</h3>
    <h4><span transition:fade={{delay:250}} style="font-size: 150px; color: orange" class="material-symbols-outlined">sync</span></h4>

{/if}