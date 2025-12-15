<script>
    import { slide } from 'svelte/transition';
    import { onMount } from "svelte";

    let toggle = $state(false);

    onMount(() => {
        setTimeout(() => {toggle = true}, 100);
    })
</script>

{#if toggle}
    <h1 transition:slide style="user-select: none;">ONBOT JAVA MANAGEMENT SYSTEM</h1>
{/if}