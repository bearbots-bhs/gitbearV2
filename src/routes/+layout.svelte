<script>
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();

	import "$lib/styles.css";

	import Navbar from "$lib/navbar.svelte";
</script>
<svelte:head>
	<title>Gitbear V2</title>
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
	<link href="https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&family=Space+Grotesk:wght@300..700&family=Space+Mono:ital,wght@0,400;0,700;1,400;1,700&display=swap" rel="stylesheet">
</svelte:head>

<Navbar />
<div style="height: 20px"></div>
{@render children()}
