<script>
	export let status;
	export let error;

	const dev = process.env.NODE_ENV === 'development';
</script>

<svelte:head>
	<title>{status}</title>
</svelte:head>

<div class='
    content
    bg-white
    max-w-6xl
    mx-auto
    px-12
    py-8
'>
    <h1 class='
        font-display
        leading-tight
        mb-4
        text-4xl
        tracking-wider
    '>{status}</h1>

    <p class='
        font-body
        text-2xl
    '>{error.message}</p>

    {#if dev && error.stack}
    	<pre>{error.stack}</pre>
    {/if}
</div>