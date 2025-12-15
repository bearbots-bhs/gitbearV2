<script>
    import { fly } from 'svelte/transition';
    import { onMount } from "svelte";

    let toggle = $state(false);

    onMount(() => {
        setTimeout(() => {toggle = true}, 100);
    })
</script>
<style>
    #navbar {
        position: fixed;
        bottom: 0;
        right: 0;
        left: 0;
        height: 70px;
        background-color: rgb(255, 255, 255);
        
        h3 {
            position: relative;
            user-select: none;
            -moz-user-select: none;
            -webkit-user-select: none;
            -ms-user-select: none;
            -webkit-user-drag: none;
        }

        h3 img {
            max-height: 50px;
            position: absolute;
            left: 10px;
            transform: translateY(-5px);
        }
        h3 span {
            position: absolute;
            right: 10px;
            right: 20px;
            transform: translateY(5px);
            color: black;
        }
    }
</style>
{#if toggle}
    <div id="navbar" transition:fly = {{y: 200, duration: 1000}}>
        <h3><img src="logo.png" alt="logo"/> <span>V2</span></h3>
    </div>
{/if}