<script>
	import favicon from '$lib/assets/favicon.ico';

	let { children } = $props();

	import "$lib/styles.css";

	import Navbar from "$lib/navbar.svelte";
</script>
<svelte:head>
	<title>Gitbear V2</title>
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
	<link href="https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&family=Space+Grotesk:wght@300..700&family=Space+Mono:ital,wght@0,400;0,700;1,400;1,700&display=swap" rel="stylesheet">
	<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200&icon_names=code_blocks,data_object,deployed_code_update,document_scanner,download,file_copy,progress_activity,sync" />
</svelte:head>

<Navbar />
<div style="height: 30px"></div>
{@render children()}
<div style="height: 70px"></div>
