<script>
    import { fly } from 'svelte/transition';
    import { onMount } from "svelte";
    import { base } from "$app/paths";

    /*
    let toggle = $state(false);

    onMount(() => {
        setTimeout(() => {toggle = true}, 100);
    })
        */

    let link = base + "/images/logo.png"
</script>
<style>
    #navbar {
        position: fixed;
        bottom: 0;
        right: 0;
        left: 0;
        height: 70px;
        background-color: rgb(255, 255, 255);
        z-index: 1000;
        
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

<div id="navbar" transition:fly = {{y: 200, duration: 1000}}>
    <h3><img src = link translate="no" alt="logo"/> <span>V2 | <a style:color="black" href="https://github.com/bearbots-bhs/gitbearV2">View Repo</a></span></h3>
</div>