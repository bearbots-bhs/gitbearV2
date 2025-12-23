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
    import { passive } from 'svelte/legacy';
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

    function goBack() {
        if (sessionStorage.getItem("returnPortal") != null) {
            window.location.href = base + "/view/" + sessionStorage.getItem("returnPortal");
        }
        else {
            window.location.href = base + "/";
        }
    }

    function scanCode(script) {
        let lines = script.split("\n");
        for (let i = 0; i < lines.length; i++) {
            lines[i] = lines[i].trim();
        }
        lines = lines.filter(line => line != "");
        //console.log(lines);

        logs.push([false, "Processing...", "Analyzing script", ""]);

        let mode = "";
        let hardwareClasses = [];
        let hardwareDevices = [];

        let error = true;
        for (let i = 0; i < lines.length; i++) {
            if (lines[i].indexOf("@TeleOp") >= 0) {
                mode = "teleop";
                error = false;
                logs.pop();
                logs.push([false, "TeleOp", "Based on this line, this should be a script for TeleOp mode. TeleOp is the 2 minute period where controllers are used by 2 players to operate the robot.", lines[i]]);

                if (lines[i].toLowerCase().indexOf("auto") >= 0) {
                    logs.push([true, "Script Name", "Is this script supposed to be TeleOp? If so, consider renaming the script to not include misleading naming. If this should be Autonomous, you need to fix this line to replace @TeleOp with @Autonomous", lines[i]])
                }
            }
            else if (lines[i].indexOf("@Autonomous") >= 0) {
                mode = "auto";
                error = false;
                logs.pop();
                logs.push([false, "Autonomous", "Based on this line, this should be a script for Autonomous mode. Autonomous is the 30 second period where robots complete tasks using pre-written code instructions, and no human interference is allowed.", lines[i]]);

                if (lines[i].toLowerCase().indexOf("teleop") >= 0 || lines[i].toLowerCase().indexOf("acompt") >= 0) {
                    logs.push([true, "Script Name", "Is this script supposed to be Autonomous? If so, consider renaming the script to not include misleading naming. If this should be TeleOp, you need to fix this line to replace @Autonomous with @TeleOp", lines[i]])
                }
            }
        }
        if (error) {
            logs.push([true, "No OpMode Declared", "No OpMode detected. Is this Autonomous or is this TeleOp? The script must have an @TeleOp or an @Autonomous"]);
            return 0;
        }
        /*
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
            */
        error = true
        for (let i = 0; i < lines.length; i++) {
            if (lines[i].indexOf("@Override")== 0) {
                error = false;
                if (lines[i+1].indexOf("public void runOpMode() {") == 0) {
                    break;
                }
                else {
                    logs.push([1, "runOpMode Function Missing", "After the @Override statement, you need to encapsulate all code that forms the base of the code instruction inside a method that overwrites void runOpMode().", lines[i]])
                }
            }
        }
        if (error) {
            logs.push([1, "No Override Statement", "All code that forms the base of the instruction needs to be inside the runOpMode() function. Since we are overwriting the contents of this function from the super class, it is good practice to include an @Override statement right before.", ""]);
        }
        error = true;
        for (let i = 0; i < lines.length; i++) {
            if (lines[i].indexOf("public class") >= 0) {
                error = false;
                if (lines[i].indexOf("extends LinearOpMode") < 0) {
                    logs.push([true, "Does not extend LinearOpMode", "This class does not extend LinearOpMode. Fix the line above.", lines[i]]);
                }
            }
        }
        if (error) {
            logs.push([true, "No Class Declaration", "Class is not declared; the above statement, which should encapsulate non-declarative code, is missing.", "public class <script-name> extends LinearOpMode {<code>}"]);
        }
        if (mode == "auto") {
            //console.log(true);
            error = true;
            for (let i = 0; i < lines.length; i++) {
                if (lines[i].trim().indexOf("waitForStart()") == 0) {
                    error = false;
                    if (lines[i+1].trim().indexOf("opModeIsActive()") >= 0) {
                        break;
                    }
                    else {
                        logs.push([1, "Active Code Failsafe Not Present", "After checking for the start button being pressed, all further non-declarative code should be encapsulated in a failsafe conditional-statement that checks the function opModeIsActive().", lines[i]])
                        break;
                    }
                }
            }
            if (error) {
                logs.push([1, "waitForStart Function Missing", "You do not have a waitForStart() function to pause between initialization and the start of the round. This is critical to have.", ""])
            }
        }
        if (mode == "teleop") {
            //console.log(lines);
            error = true;
            for (let i = 0; i < lines.length; i++) {
                if (lines[i].trim().indexOf("waitForStart()") == 0) {
                    error = false;
                    if (lines[i+1].trim().indexOf("while (opModeIsActive()") >= 0) {
                        break;
                    }
                    else {
                        logs.push([1, "Code Looping Not Present", "After checking for the start button being pressed, all game controller actions are only processed for a fraction of a second. The code that follows needs to be encapsulated in a while-loop that checks the function opModeIsActive(). <br><br>*Ignore this if you are calibrating after waitForStart().", lines[i]])
                        break;
                    }
                }
            }
            if (error) {
                logs.push([1, "waitForStart Function Missing", "You do not have a waitForStart() function to pause between initialization and the start of TeleOp. This is critical to have.", ""])
            }
        }
        error = true;
        for (let i = 0; i < lines.length; i++) {
            if (lines[i].indexOf("import com.qualcomm.robotcore.hardware.") == 0) {
                let hardware = lines[i].substring(39);
                hardware = hardware.substring(0, hardware.length-1);
                //console.log(hardware);
                hardwareClasses.push(hardware);
            }
            else if (lines[i].indexOf("public class") == 0) {
                let hardwareLabel = "";
                for (let i in hardwareClasses) {
                    hardwareLabel += hardwareClasses[i] + ", ";
                }
                hardwareLabel = hardwareLabel.trim();
                hardwareLabel = hardwareLabel.substring(0, hardwareLabel.length-1);
                logs.push([0, "Hardware Classes", "Based on scanning, these are all hardware types in use. If this list is missing a hardware type, you have not imported the type. If one of these types is not in use, consider removing the import statement to clean up your code.", hardwareLabel]);
                break; 
            }
        }
        for (let i = 0; i < lines.length; i++) {
            if (lines[i].indexOf("private") == 0) {
                let process = lines[i].substring(7).trim();
                process = process.substring(0, process.length-1);
                let processHardware = "";
                let processName = ""
                for (let x = 0; x < process.length; x++) {
                    if (process[x] == ' ') {
                        break;
                    }
                    else {
                        processHardware += process[x]
                    }
                }
                if (hardwareClasses.indexOf(processHardware) >= 0) {
                    if (processName.indexOf("(") < 0 && processName.indexOf(")") < 0) {
                        processName = process.substring(processHardware.length).trim();
                        hardwareDevices.push([processName, processHardware, 0]);
                    }
                    else {
                        logs.push([0, "Function that returns a Hardware Device", "Is this a function that returns a hardware device, or is this line trying to declare a hardware device? Based on scanning, this is currently a function declaration.", lines[i]])
                    }
                }
                else if (["String", "int", "double", "boolean", "void", "ElapsedTime"].indexOf(processHardware) < 0) {
                    logs.push([1, "Undetermined Hardware Type", "Scanner is unable to find the hardware type used to declare this device. If this is a device, make sure that the type is imported at the start of the script.", lines[i]])
                }
            }
        }
        logs.push([0, "Hardware Devices", `Based on scanning, this script uses a total of ${hardwareDevices.length} hardware devices.`, ""]);
        for (let i in hardwareClasses) {
            let x = hardwareClasses[i];
            let count = 0;

            for (let k in hardwareDevices) {
                if (x == hardwareDevices[k][1]) {
                    count++;
                }
            }

            if (count == 0) {
                logs.push([1, "Unused Hardware Class", `${x} hardware class is imported; however, no hardware devices use this class. If you don't plan to eventually use this class, remove its import statement.`, x])
            }
        }
        for (let i = 0; i < lines.length; i++) {
            if (lines[i].indexOf("hardwareMap.get(") > 0) {
                let process = "";
                let name = "";
                let hardwareClass = "";
                for (let k = 0; k < lines[i].length; k++) {
                    if (lines[i].substring(k, k+2) == " =") {
                        //console.log(name);
                        break;
                    }
                    else {
                        name += lines[i].substring(k, k+1);
                    }
                }
                let x;
                for (x = 0; x < lines[i].length; x++) {
                    if (process.indexOf("hardwareMap.get(") > 0) {
                        break;
                    }
                    else {
                        process += lines[i].substring(x, x+1);
                    }
                }
                for (x; x < lines[i].length; x++) {
                    if (lines[i].substring(x, x+1) == ".") {
                        //console.log(hardwareClass);
                        break;
                    }
                    else {
                        hardwareClass += lines[i].substring(x, x+1);
                    }
                }
                let notFound = true;
                for (let k = 0; k < hardwareDevices.length; k++) {
                    if (hardwareDevices[k][0] == name && hardwareDevices[k][1] == hardwareClass) {
                        hardwareDevices[k][2] = 1;
                        notFound = false;
                        break;
                    }
                }
                if (notFound) {
                    logs.push([1, "Hardware Device Mapped but not Declared", `${hardwareClass} ${name} is mapped using the hardwareMap.get() function; however, it has not been declared as a private variable outside the runOpMode() function`, name])
                }
            }
        }
        for (let i = 0; i < hardwareDevices.length; i++) {
            let device = hardwareDevices[i];
            if (!device[2]) {
                logs.push([1, "Hardware Device Not Mapped", `${device[1]} ${device[0]} is declared; however, it is not mapped to a port using the hardwareMap.get() function.`, device[0]])
            }
        }

        logs.push([0, "Finished", "Scanning is now finished; there should be no further critical errors. If you are running into further issues, consult other sources or use debugging strategies to solve your issue!", ""]);
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

    #codeDisplay {
        vertical-align: top;
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
            background-color: rgba(90, 29, 18, 1);
            div {
                background-color: rgb(139, 98, 98);
            }
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
    <h3 style="margin-top: 10px; margin-bottom: 50px" transition:fade={{delay: 100}}><button id="returnButton" onclick = {goBack}>Return</button></h3>
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
                            <h5>{@html x[2]}</h5>
                        </div>
                    {/each}
                </td>
            </tr>
        </tbody>
    </table>
{/if}