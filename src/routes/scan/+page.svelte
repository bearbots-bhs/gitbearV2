<script>
    import { fade, fly, slide } from 'svelte/transition';
    import { onMount } from "svelte";
    import { base } from "$app/paths";
    import { page } from "$app/stores";

    let loading = $state(true); // Used to show or hide the loading icon
    

    let toggle = $state(false); // Used for intro transition

    let script = $state(""); // Code for script, without task 
    let code = $state("");

    let logs = $state([]); // Feedback or error detection logs


    import hljs from 'highlight.js/lib/core';
    import java from 'highlight.js/lib/languages/java'
    hljs.registerLanguage("java", java);


    onMount(() => {
        loading = true;
        let script = sessionStorage.getItem("scanThis");
        if (script == null) {
            window.location.href = base + "/view"
        }
    
        //console.log(script);
        code = script;
        code = hljs.highlight(code, { language: "java" }).value;
        scanCode(script);
    });

    onMount(() => {
        setTimeout(() => {toggle = true}, 100)
    })

    function scanCode(script) {
        let lines = script.split("\n");
        for (let i = 0; i < lines.length; i++) {
            lines[i] = lines[i].trim();
        }
        lines = lines.filter(line => line != "");
        //console.log(lines);

        logs.push([false, "Processing...", "Analyzing script", ""]);

        let mode = "";

        let error = true;
        for (let i = 0; i < lines.length; i++) {
            if (lines[i].indexOf("@TeleOp") >= 0) {
                mode = "teleop";
                error = false;
                logs.pop();
                logs.push([false, "TeleOp", "Based on this line, this should be a script for TeleOp mode. TeleOp is the 2 minute period where controllers are used by 2 players to operate the robot.", lines[i]]);

                if (lines[i].toLowerCase().indexOf("auto") >= 0) {
                    logs.push([true, "Warning based on Script Name", "Is this script supposed to be TeleOp? If so, consider renaming the script to not include misleading naming. If this should be Autonomous, you need to fix this line to replace @TeleOp with @Autonomous", lines[i]])
                }
            }
            else if (lines[i].indexOf("@Autonomous") >= 0) {
                mode = "auto";
                error = false;
                logs.pop();
                logs.push([false, "Autonomous", "Based on this line, this should be a script for Autonomous mode. Autonomous is the 30 second period where robots complete tasks using pre-written code instructions, and no human interference is allowed.", lines[i]]);

                if (lines[i].toLowerCase().indexOf("teleop") >= 0 || lines[i].toLowerCase().indexOf("acompt") >= 0) {
                    logs.push([true, "Warning based on Script Name", "Is this script supposed to be Autonomous? If so, consider renaming the script to not include misleading naming. If this should be TeleOp, you need to fix this line to replace @Autonomous with @TeleOp", lines[i]])
                }
            }
        }
        if (error) {
            logs.push([true, "Critical Error", "No OpMode detected. Is this Autonomous or is this TeleOp? The script must have an @TeleOp or an @Autonomous"]);
            return 0;
        }
        error = true;
        for (let i = 0; i < lines.length; i++) {
            if (mode == "teleop" && lines[i] == "import com.qualcomm.robotcore.eventloop.opmode.TeleOp;") {
                error = false;
                break;
            }
            else if (mode == "auto" && lines[i] == "import com.qualcomm.robotcore.eventloop.opmode.Autonomous;") {
                error = false;
                break;
            }
        }
        if (error) {
            logs.push([true, "Critical Error", "No OpMode imported. The script must have an import statement, such as modelled above.", "import com.qualcomm.robotcore.eventloop.opmode.Autonomous;"]);
        }
        error = true;
        for (let i = 0; i < lines.length; i++) {
            if (lines[i].indexOf("public class") >= 0) {
                error = false;
                if (lines[i].indexOf("extends LinearOpMode") < 0) {
                    logs.push([true, "Critical Error", "This class does not extend LinearOpMode. Fix the line above.", lines[i]]);
                }
            }
        }
        if (error) {
            logs.push([true, "Critical Error", "Class is not declared; the above statement, which should encapsulate non-declarative code, is missing.", "public class <script-name> extends LinearOpMode {<code>}"]);
            loading = false;
        }
        error = true;

        logs.push([false, "Finished", "Scanning is now finished; there should be no further critical errors. If you are running into further issues, consult other sources or use debugging strategies to solve your issue!", ""]);
        loading = false;
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
        width: 50em;
        background-color: rgb(60, 62, 94);
        border-radius: 20px;
        margin-right: 20px;
        margin-left: 20px;
        text-align: left;
        padding: 10px;
        padding-left: 40px;
        position: relative;
        overflow-x: scroll;

        pre {
            white-space: pre;
            margin: 0;
            text-align: left;
        }


   
     
    }
    #logsDisplay { 
        position: sticky;
        top: 200px;
        width: 43em;
        vertical-align: top;

        div {
            padding: 20px;
            margin: 20px;
            background-color: rgb(68, 79, 94);
            border-radius: 20px;
            
            div {
                background-color: grey;
                padding: 3px;
                padding-left: 6px;
                padding-right: 6px;
            }
        } 
        div.fail {
            background-color: rgb(100, 29, 17);
        }
    }

    #loadingContainer {
        position: fixed;
        top: 5px;
        left: 15px;
        color: white;
        z-index: 1000;
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
    <h1 transition:slide style="user-select: none;">SCANNER</h1>
    <h3 style="margin-top: 10px; margin-bottom: 50px" transition:fade={{delay: 100}}><button id="returnButton" onclick = {() => {window.location.href = base + "/"}}>Return</button></h3>
    {#if loading}
        <div transition:fade id="loadingContainer">
            <h4><span class="material-symbols-outlined">progress_activity</span></h4>
        </div>
    {/if}
    <table>
        <tbody>
            <tr id="generalDisplay">
                <td id="codeDisplay">
                    <div>
                        <h4><pre><code>{@html code}</code></pre></h4>
                    </div>
                </td>
                <td id="logsDisplay">
                    {#each logs as x}
                        <div class:fail={x[0]}>
                            <h3>{x[1]}</h3>
                            {#if x[3].length != 0}<div><h4><code>{x[3]}</code></h4></div>{/if}
                            <h5>{x[2]}</h5>
                        </div>
                    {/each}
                </td>
            </tr>
        </tbody>
    </table>
{/if}