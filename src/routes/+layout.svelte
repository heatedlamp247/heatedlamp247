<script lang="ts">
	import "./layout.css";
	import "iconify-icon";

	let { children } = $props();
</script>

<svelte:head>
	<title>Jared's Portfolio</title>
</svelte:head>

{@render children()}
