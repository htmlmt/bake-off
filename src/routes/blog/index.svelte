<script context="module">
	export function preload({ params, query }) {
		return this.fetch(`blog.json`).then(r => r.json()).then(posts => {
			return { posts };
		});
	}
</script>

<script>
	export let posts;
</script>

<svelte:head>
	<title>Blog</title>
</svelte:head>

<div class='
    bg-white
    content
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
    '>Blog</h1>

    <ul>
    	{#each posts as post}
    		<li>
                <a class='
                    font-body
                    text-2xl
                ' rel='prefetch' href='blog/{post.slug}'>{post.title}</a>
            </li>
    	{/each}
    </ul>
</div>