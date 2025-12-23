<script>
    import { fade, fly, slide } from 'svelte/transition';
    import { onMount } from "svelte";
    import { base } from "$app/paths";
    import { page } from "$app/stores";

    let loading = $state(true); // Used to show or hide the loading icon
    

    let toggle = $state(false); // Used for intro transition

    let title = $state("Loading..."); // Header
    let scripts = $state([]); // Contains JSON info for all scripts

    let code = $state("// Select a script to get started //") // Displayed script
    let selected = $state(""); // Filename for selected script
    let script = $state(""); // Code for script, without task 

    let force = $state(""); // Used to sync files

    import hljs from 'highlight.js/lib/core';
    import java from 'highlight.js/lib/languages/java'
    hljs.registerLanguage("java", java);


    onMount(async() => {
        setTimeout(() => {toggle = true}, 100);
        let year = $page.params.slug;

        let configRaw = await fetch(`https://raw.githubusercontent.com/bearbots-bhs/${year}/refs/heads/main/config.json`);
        if (!configRaw.ok) {
            window.location.href = base + "/view";
        }
        let config = await configRaw.json();
        
        title = config[0].name;
        scripts = config[1];
        loading = false;

    });

    async function displayCode(fileName, task) {
        loading = true;
        let scriptRaw = await fetch(`https://raw.githubusercontent.com/bearbots-bhs/${$page.params.slug}/refs/heads/main/${fileName}${force}`)
        
        script = await scriptRaw.text();
        //console.log(script);
        code = "// " + task + "\n\n\n\n" + script;
        code = hljs.highlight(code, { language: "java" }).value;
        selected = fileName;
        loading = false;
    }

    async function copyCode() {
        if (selected == "" || script.length <= 0) {
            return;
        }
        await navigator.clipboard.writeText(script);
    }

    function forceGen() {
        let time = new Date();
        force = "?sync=" + time.getTime();

        script = "";
        selected = "";
        code = "// Files should now be synced; if not, wait a few minutes before trying again. Click buttons to load scripts //"
    }

    function scanCode() {
        sessionStorage.setItem("scanThis", script);
        //console.log(sessionStorage.getItem("scanThis"));
        window.location.href = base + "/scan";
    }
 
</script>
<style>
    #returnButton {
        font-family: Space Grotesk, Space Mono, Montserrat, Futura;
        font-size: 15px;
        padding: 10px;
    }
    #returnButton:hover {
        box-shadow: 0px 0px 5px 10px rgba(62, 62, 165, 0.308);
    }

    #codeDisplay div {
        width: 71em;
        height: 28em;
        background-color: rgb(60, 62, 94);
        border-radius: 20px;
        overflow-y: scroll;
        margin-right: 20px;
        margin-left: 20px;
        text-align: left;
        padding: 10px;
        padding-left: 40px;
        position: relative;

        pre {
            white-space: pre;
            margin: 0;
            text-align: left;
        }

        #copyButton {
            position: absolute;
            top: 15px;
            right: 15px;
            transition: transform 0.1s ease-in, box-shadow 0.3s ease-in-out;
        }
        #copyButton:hover {
            box-shadow: 0px 0px 3px 6px rgba(105, 105, 148, 0.432);
        }
        #copyButton:active {
            transform: scale(0.9)
        }

        #scanButton {
            position: absolute;
            top: 15px;
            right: 68px;
            transition: transform 0.1s ease-in, box-shadow 0.3s ease-in-out;
        }
        #scanButton:hover {
            box-shadow: 0px 0px 3px 6px rgba(105, 105, 148, 0.432);
        }
        #scanButton:active {
            transform: scale(0.9)
        }
     
    }
    #scriptsDisplay {
        position: sticky;
        top: 200px;

        button {
            font-family: Space Grotesk, Space Mono, Montserrat, Futura;
            font-size: 15px;
            padding: 10px;
        }
        button:hover {
            box-shadow: 0px 0px 5px 10px rgba(62, 62, 165, 0.432);
        }
        button.selected {
            background-color: rgb(223, 97, 24);
            box-shadow: 0px 0px 5px 10px rgba(165, 100, 62, 0.267);
        }
        button.selected:hover {
            box-shadow: 0px 0px 5px 10px rgba(165, 100, 62, 0.267);
        }
    }
    #forceButton {
        position: fixed;
        right: 15px;
        top: 15px;
    }
    #forceButton:hover {
        box-shadow: 0px 0px 3px 6px rgba(105, 105, 148, 0.432);
    }
    #repoButton {
        position: fixed;
        right: 70px;
        top: 15px;
    }
    #repoButton:hover {
        box-shadow: 0px 0px 3px 6px rgba(105, 105, 148, 0.432);
    }

    #loadingContainer {
        position: fixed;
        top: 5px;
        left: 15px;
        color: white;

        span {
            animation: loading 0.8s infinite ease-in;
        }
    }

    @keyframes loading {
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
 <svelte:head>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/a11y-dark.min.css">
</svelte:head>

{#if toggle}
    <h1 transition:slide style="user-select: none; font-weight: 700">{title}</h1>
    <h3 style="margin-top: 10px; margin-bottom: 50px" transition:fade={{delay: 100}}><button id="returnButton" onclick = {() => {window.location.href = base + "/"}}>Return</button></h3>
    {#if loading}
        <div transition:fade id="loadingContainer">
            <h4><span class="material-symbols-outlined">progress_activity</span></h4>
        </div>
    {/if}
    <h3><button id="repoButton" onclick={() => {window.location.href = `https://github.com/bearbots-bhs/${$page.params.slug}`}} transition:fly={{y:-200, delay:100}}><span translate="no" title="View GitHub Repository" class="material-symbols-outlined">data_object</span></button> <button onclick = {forceGen} transition:fly={{y:-200, delay: 200}} id="forceButton" title="Sync Recent Changes"><span translate = "no" class="material-symbols-outlined">deployed_code_update</span></button></h3>
    <table>
        <tbody>
            <tr id="generalDisplay">
                <td id="codeDisplay">
                    <div>
                        <h4><pre><code>{@html code}</code></pre></h4>
                        {#if script.length > 0}<button title="Copy Script Contents" in:fly={{y:50, delay:200}} out:fade onclick = {copyCode} id="copyButton"><span translate = "no" class="material-symbols-outlined">file_copy</span></button><button title="Scan Script Contents" in:fly={{y:50, delay:100}} out:fade onclick = {scanCode} id="scanButton"><span translate = "no" class="material-symbols-outlined">document_scanner</span></button>{/if}
                    </div>
                </td>
                <td id="scriptsDisplay">
                    {#each scripts as x}
                        <h4><button class:selected={selected == x.name} onclick = {() => {displayCode(x.name, x.task)}}>{x.name}</button></h4>
                    {/each}
                </td>
            </tr>
        </tbody>
    </table>
{/if}