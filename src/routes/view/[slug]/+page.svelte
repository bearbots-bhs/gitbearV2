<script>
    import { fade, fly, slide } from 'svelte/transition';
    import { onMount } from "svelte";
    import { base } from "$app/paths";

    import { page } from "$app/stores";
    

    let toggle = $state(false);

    let title = $state("Loading...");
    let scripts = $state([]);

    let code = $state("// Select a script to get started //")
    let selected = $state("");

    let script = $state("");

    let force = $state("");


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

    });

    async function displayCode(fileName, task) {
        let scriptRaw = await fetch(`https://raw.githubusercontent.com/bearbots-bhs/${$page.params.slug}/refs/heads/main/${fileName}${force}`)
        
        script = await scriptRaw.text();
        //console.log(script);
        code = "// " + task + "\n\n\n\n" + script;
        selected = fileName;
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
        width: 72em;
        height: 28em;
        background-color: rgb(60, 62, 97);
        border-radius: 20px;
        overflow-y: scroll;
        margin-right: 20px;
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
</style>

{#if toggle}
    <h1 transition:slide style="user-select: none;">{title}</h1>
    <h3 style="margin-top: 10px; margin-bottom: 50px" transition:fade={{delay: 100}}><button id="returnButton" onclick = {() => {window.location.href = base + "/"}}>Return</button></h3>
    <h3><button id="repoButton" onclick={() => {window.location.href = `https://github.com/bearbots-bhs/${$page.params.slug}`}} transition:fly={{y:-200, delay:100}}><span translate="no" title="View GitHub Repository" class="material-symbols-outlined">data_object</span></button> <button onclick = {forceGen} transition:fly={{y:-200, delay: 200}} id="forceButton" title="Sync Recent Changes"><span translate = "no" class="material-symbols-outlined">deployed_code_update</span></button></h3>
    <table>
        <tbody>
            <tr id="generalDisplay">
                <td id="codeDisplay">
                    <div>
                        <h4><pre><code>{code}</code></pre></h4>
                        {#if script.length > 0}<button title="Copy Script Contents" transition:fly={{y:100}} onclick = {copyCode} id="copyButton"><span translate = "no" class="material-symbols-outlined">file_copy</span></button>{/if}
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